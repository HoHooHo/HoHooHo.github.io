# NewObject和CreateDefaultSubobject
----------------------


!!! info "在 `UE4` 中，创建 `UObject` 对象有 `2` 种方式"
    - `NewObject`
        - 在非构造函数中使用
    - `CreateDefaultSubobject`
        - 在构造函数中使用



## NewObject

`NewObject` 构造完参数 `FStaticConstructObjectParameters` 后，直接调用的 `StaticConstructObject_Internal(Params)`

```c++
/**
 * Convenience template for constructing a gameplay object
 *
 * @param	Outer		the outer for the new object.  If not specified, object will be created in the transient package.
 * @param	Class		the class of object to construct
 * @param	Name		the name for the new object.  If not specified, the object will be given a transient name via MakeUniqueObjectName
 * @param	Flags		the object flags to apply to the new object
 * @param	Template	the object to use for initializing the new object.  If not specified, the class's default object will be used
 * @param	bCopyTransientsFromClassDefaults	if true, copy transient from the class defaults instead of the pass in archetype ptr (often these are the same)
 * @param	InInstanceGraph						contains the mappings of instanced objects and components to their templates
 * @param	ExternalPackage						Assign an external Package to the created object if non-null
 *
 * @return	a pointer of type T to a new object of the specified class
 */

template< class T >
FUNCTION_NON_NULL_RETURN_START
	T* NewObject(UObject* Outer, const UClass* Class, FName Name = NAME_None, EObjectFlags Flags = RF_NoFlags, UObject* Template = nullptr, bool bCopyTransientsFromClassDefaults = false, FObjectInstancingGraph* InInstanceGraph = nullptr, UPackage* ExternalPackage = nullptr)
FUNCTION_NON_NULL_RETURN_END
{
	if (Name == NAME_None)
	{
		FObjectInitializer::AssertIfInConstructor(Outer, TEXT("NewObject with empty name can't be used to create default subobjects (inside of UObject derived class constructor) as it produces inconsistent object names. Use ObjectInitializer.CreateDefaultSubobject<> instead."));
	}

#if DO_CHECK
	// Class was specified explicitly, so needs to be validated
	CheckIsClassChildOf_Internal(T::StaticClass(), Class);
#endif

	FStaticConstructObjectParameters Params(Class);
	Params.Outer = Outer;
	Params.Name = Name;
	Params.SetFlags = Flags;
	Params.Template = Template;
	Params.bCopyTransientsFromClassDefaults = bCopyTransientsFromClassDefaults;
	Params.InstanceGraph = InInstanceGraph;
	Params.ExternalPackage = ExternalPackage;
	return static_cast<T*>(StaticConstructObject_Internal(Params));
}
```

## CreateDefaultSubobject

`CreateDefaultSubobject` 最终也是调用 `StaticConstructObject_Internal` 来创建对象，但在这之前会经过 `FObjectInitializer::CreateDefaultSubobject` 的处理

```c++
	/**
	 * Create an optional component or subobject. Optional subobjects will not get created
	 * if a derived class specified DoNotCreateDefaultSubobject with the subobject's name.
	 * @param	TReturnType					Class of return type, all overrides must be of this type
	 * @param	SubobjectName				Name of the new component
	 * @param	bTransient					True if the component is being assigned to a transient property. This does not make the component itself transient, but does stop it from inheriting parent defaults
	 */
	template<class TReturnType>
	TReturnType* CreateOptionalDefaultSubobject(FName SubobjectName, bool bTransient = false)
	{
		UClass* ReturnType = TReturnType::StaticClass();
		return static_cast<TReturnType*>(CreateDefaultSubobject(SubobjectName, ReturnType, ReturnType, /*bIsRequired =*/ false, bTransient));
	}
	
	/**
	 * Create an optional component or subobject. Optional subobjects will not get created
	 * if a derived class specified DoNotCreateDefaultSubobject with the subobject's name.
	 * @param	TReturnType					Class of return type, all overrides must be of this type
	 * @param	TClassToConstructByDefault	Class of object to actually construct, must be a subclass of TReturnType
	 * @param	SubobjectName				Name of the new component
	 * @param	bTransient					True if the component is being assigned to a transient property. This does not make the component itself transient, but does stop it from inheriting parent defaults
	 */
	template<class TReturnType, class TClassToConstructByDefault>
	TReturnType* CreateOptionalDefaultSubobject(FName SubobjectName, bool bTransient = false)
	{
		return static_cast<TReturnType*>(CreateDefaultSubobject(SubobjectName, TReturnType::StaticClass(), TClassToConstructByDefault::StaticClass(), /*bIsRequired =*/ false, bTransient));
	}
```

```c++
/** Utility function for templates below */
UObject* CreateDefaultSubobject(FName SubobjectFName, UClass* ReturnType, UClass* ClassToCreateByDefault, bool bIsRequired, bool bIsTransient);

UObject* UObject::CreateDefaultSubobject(FName SubobjectFName, UClass* ReturnType, UClass* ClassToCreateByDefault, bool bIsRequired, bool bIsTransient)
{
	FObjectInitializer* CurrentInitializer = FUObjectThreadContext::Get().TopInitializer();
	UE_CLOG(!CurrentInitializer, LogObj, Fatal, TEXT("No object initializer found during construction."));
	UE_CLOG(CurrentInitializer->Obj != this, LogObj, Fatal, TEXT("Using incorrect object initializer."));
	return CurrentInitializer->CreateDefaultSubobject(this, SubobjectFName, ReturnType, ClassToCreateByDefault, bIsRequired, bIsTransient);
}
```


### FObjectInitializer::CreateDefaultSubobject

主要目的就是 使用 `ComponentOverrides` 中的指定 `UClass` 替代 `ClassToCreateByDefault` 来创建对象

- 会检查是否在构造函数中调用
- 通过 `ObjectInitializer->SetDefaultSubobjectClass` 来指定 `UClass` 替代 `ClassToCreateByDefault`
- 通过 `ObjectInitializer->DoNotCreateDefaultSubobject` 指定为 `nullptr` 且 `!bIsRequired`，则不创建

``` c++
UObject* FObjectInitializer::CreateDefaultSubobject(UObject* Outer, FName SubobjectFName, UClass* ReturnType, UClass* ClassToCreateByDefault, bool bIsRequired, bool bIsTransient) const
{
	UE_CLOG(!FUObjectThreadContext::Get().IsInConstructor, LogClass, Fatal, TEXT("Subobjects cannot be created outside of UObject constructors. UObject constructing subobjects cannot be created using new or placement new operator."));
	if (SubobjectFName == NAME_None)
	{
		UE_LOG(LogClass, Fatal, TEXT("Illegal default subobject name: %s"), *SubobjectFName.ToString());
	}

	UObject* Result = NULL;
	UClass* OverrideClass = ComponentOverrides.Get(SubobjectFName, ReturnType, ClassToCreateByDefault, *this);
	if (!OverrideClass && bIsRequired)
	{
		OverrideClass = ClassToCreateByDefault;
		UE_LOG(LogClass, Warning, TEXT("Ignored DoNotCreateDefaultSubobject for %s as it's marked as required. Creating %s."), *SubobjectFName.ToString(), *OverrideClass->GetName());
	}
	if (OverrideClass)
	{
		check(OverrideClass->IsChildOf(ReturnType));

		if (OverrideClass->HasAnyClassFlags(CLASS_Abstract))
		{
			// Attempts to create an abstract class will return null. If it is not optional or the owning class is not also abstract report a warning.
			if (!bIsRequired && !Outer->GetClass()->HasAnyClassFlags(CLASS_Abstract))
			{
				UE_LOG(LogClass, Warning, TEXT("Required default subobject %s not created as requested class %s is abstract. Returning null."), *SubobjectFName.ToString(), *OverrideClass->GetName());
			}
		}
		else
		{
			UObject* Template = OverrideClass->GetDefaultObject(); // force the CDO to be created if it hasn't already
			EObjectFlags SubobjectFlags = Outer->GetMaskedFlags(RF_PropagateToSubObjects) | RF_DefaultSubObject;
			bool bOwnerArchetypeIsNotNative;
			UClass* OuterArchetypeClass;

			// It is not safe to mark this component as properly transient, that results in it being nulled incorrectly

			OuterArchetypeClass = Outer->GetArchetype()->GetClass();
			bOwnerArchetypeIsNotNative = !OuterArchetypeClass->HasAnyClassFlags(CLASS_Native | CLASS_Intrinsic);

			const bool bOwnerTemplateIsNotCDO = ObjectArchetype != nullptr && ObjectArchetype != Outer->GetClass()->GetDefaultObject(false) && !Outer->HasAnyFlags(RF_ClassDefaultObject);
#if !UE_BUILD_SHIPPING
			// Guard against constructing the same subobject multiple times.
			// We only need to check the name as ConstructObject would fail anyway if an object of the same name but different class already existed.
			if (ConstructedSubobjects.Find(SubobjectFName) != INDEX_NONE)
			{
				UE_LOG(LogClass, Fatal, TEXT("Default subobject %s %s already exists for %s."), *OverrideClass->GetName(), *SubobjectFName.ToString(), *Outer->GetFullName());
			}
			else
			{
				ConstructedSubobjects.Add(SubobjectFName);
			}
#endif
			FStaticConstructObjectParameters Params(OverrideClass);
			Params.Outer = Outer;
			Params.Name = SubobjectFName;
			Params.SetFlags = SubobjectFlags;

			Result = StaticConstructObject_Internal(Params);
			if (!bIsTransient && (bOwnerArchetypeIsNotNative || bOwnerTemplateIsNotCDO))
			{
				UObject* MaybeTemplate = nullptr;
				if (bOwnerTemplateIsNotCDO)
				{
					// Try to get the subobject template from the specified object template
					MaybeTemplate = ObjectArchetype->GetDefaultSubobjectByName(SubobjectFName);
				}
				if (!MaybeTemplate)
				{
					// The archetype of the outer is not native, so we need to copy properties to the subobjects after the C++ constructor chain for the outer has run (because those sets properties on the subobjects)
					MaybeTemplate = OuterArchetypeClass->GetDefaultSubobjectByName(SubobjectFName);
				}
				if (MaybeTemplate && MaybeTemplate->IsA(ReturnType) && Template != MaybeTemplate)
				{
					ComponentInits.Add(Result, MaybeTemplate);
				}
			}
			if (Outer->HasAnyFlags(RF_ClassDefaultObject) && Outer->GetClass()->GetSuperClass())
			{
#if WITH_EDITOR
				// Default subobjects on the CDO should be transactional, so that we can undo/redo changes made to those objects.
				// One current example of this is editing natively defined components in the Blueprint Editor.
				Result->SetFlags(RF_Transactional);
#endif
				Outer->GetClass()->AddDefaultSubobject(Result, ReturnType);
			}
			// Clear PendingKill flag in case we recycled a subobject of a dead object.
			// @todo: we should not be recycling subobjects unless we're currently loading from a package
			Result->ClearInternalFlags(EInternalObjectFlags::PendingKill);
		}
	}
	return Result;
}
```


## UObject* StaticConstructObject_Internal

在函数 `StaticConstructObject_Internal`  中

  - `Result = StaticAllocateObject(InClass, InOuter, InName, InFlags, Params.InternalSetFlags, bCanRecycleSubobjects, &bRecycledSubobject, Params.ExternalPackage)`;
      - 申请内存空间
  - `(*InClass->ClassConstructor)( FObjectInitializer(Result, InTemplate, Params.bCopyTransientsFromClassDefaults, true, Params.InstanceGraph) )`;
      - 调用构造函数

``` c++
UObject* StaticConstructObject_Internal(const FStaticConstructObjectParameters& Params)
{
    ....
    // 申请内存
    Result = StaticAllocateObject(InClass, InOuter, InName, InFlags, Params.InternalSetFlags, bCanRecycleSubobjects, &bRecycledSubobject, Params.ExternalPackage);
    check(Result != NULL);
    // Don't call the constructor on recycled subobjects, they haven't been destroyed.
    if (!bRecycledSubobject)
    {
        STAT(FScopeCycleCounterUObject ConstructorScope(InClass->GetFName().IsNone() ? nullptr : InClass, GET_STATID(STAT_ConstructObject)));
        //调用构造函数
        (*InClass->ClassConstructor)( FObjectInitializer(Result, InTemplate, Params.bCopyTransientsFromClassDefaults, true, Params.InstanceGraph) );
    }
    
    if( GIsEditor && GUndo && (InFlags & RF_Transactional) && !(InFlags & RF_NeedLoad) && !InClass->IsChildOf(UField::StaticClass()) )
    {
        // Set RF_PendingKill and update the undo buffer so an undo operation will set RF_PendingKill on the newly constructed object.
        Result->MarkPendingKill();
        SaveToTransactionBuffer(Result, false);
        Result->ClearPendingKill();
    }
    return Result;
}
```