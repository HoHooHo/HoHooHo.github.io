# 宏GENERATED_XX_BODY
----------------------

UE4中定义了多种 `GENERATED_XX_BODY` 的宏，看宏定义最终只有 `GENERATED_BODY` 和 `GENERATED_BODY_LEGACY` 两种，而区别只是构造函数是无参构造函数还是带有参数 `const FObjectInitializer& X` 的构造函数

- 默认无参构造函数
    - `GENERATED_BODY`
	- `GENERATED_USTRUCT_BODY`
- 带有参数 `const FObjectInitializer& X` 的构造函数
    - `GENERATED_UCLASS_BODY`
    - `GENERATED_UINTERFACE_BODY`
    - `GENERATED_IINTERFACE_BODY`
    - `GENERATED_BODY_LEGACY`

而 `GENERATED_BODY` 和 `GENERATED_BODY_LEGACY` 最终也是指向宏 `BODY_MACRO_COMBINE_INNER`，只是后缀不同

```c++
// This pair of macros is used to help implement GENERATED_BODY() and GENERATED_USTRUCT_BODY()
#define BODY_MACRO_COMBINE_INNER(A,B,C,D) A##B##C##D
#define BODY_MACRO_COMBINE(A,B,C,D) BODY_MACRO_COMBINE_INNER(A,B,C,D)

// Include a redundant semicolon at the end of the generated code block, so that intellisense parsers can start parsing
// a new declaration if the line number/generated code is out of date.
#define GENERATED_BODY_LEGACY(...) BODY_MACRO_COMBINE(CURRENT_FILE_ID,_,__LINE__,_GENERATED_BODY_LEGACY);
#define GENERATED_BODY(...) BODY_MACRO_COMBINE(CURRENT_FILE_ID,_,__LINE__,_GENERATED_BODY);

#define GENERATED_USTRUCT_BODY(...) GENERATED_BODY()
#define GENERATED_UCLASS_BODY(...) GENERATED_BODY_LEGACY()
#define GENERATED_UINTERFACE_BODY(...) GENERATED_BODY_LEGACY()
#define GENERATED_IINTERFACE_BODY(...) GENERATED_BODY_LEGACY()
```


## BODY_MACRO_COMBINE_INNER

- `#define BODY_MACRO_COMBINE_INNER(A,B,C,D) A##B##C##D`
    - 通过传入的字符串组合成一个新的宏，对应的宏定义在 `"XX.generated.h"` 头文件中
    - `"XX.generated.h"` 可以在 `MyProject/Intermediate/Build/Win64` 目录下找到

在 `"XX.generated.h"` 的最下方可以看到 `#define BODY_MACRO_COMBINE_INNER(A,B,C,D) A##B##C##D` 生成的字符串对应的宏定义

```c++
#define MyProject_Source_MyProject_Public_MyPlayerState_h_16_GENERATED_BODY_LEGACY \
PRAGMA_DISABLE_DEPRECATION_WARNINGS \
public: \
	MyProject_Source_MyProject_Public_MyPlayerState_h_16_PRIVATE_PROPERTY_OFFSET \
	MyProject_Source_MyProject_Public_MyPlayerState_h_16_SPARSE_DATA \
	MyProject_Source_MyProject_Public_MyPlayerState_h_16_RPC_WRAPPERS \
	MyProject_Source_MyProject_Public_MyPlayerState_h_16_INCLASS \
	MyProject_Source_MyProject_Public_MyPlayerState_h_16_STANDARD_CONSTRUCTORS \
public: \
PRAGMA_ENABLE_DEPRECATION_WARNINGS

#define MyProject_Source_MyProject_Public_MyPlayerState_h_16_GENERATED_BODY \
PRAGMA_DISABLE_DEPRECATION_WARNINGS \
public: \
	MyProject_Source_MyProject_Public_MyPlayerState_h_16_PRIVATE_PROPERTY_OFFSET \
	MyProject_Source_MyProject_Public_MyPlayerState_h_16_SPARSE_DATA \
	MyProject_Source_MyProject_Public_MyPlayerState_h_16_RPC_WRAPPERS_NO_PURE_DECLS \
	MyProject_Source_MyProject_Public_MyPlayerState_h_16_INCLASS_NO_PURE_DECLS \
	MyProject_Source_MyProject_Public_MyPlayerState_h_16_ENHANCED_CONSTRUCTORS \
private: \
PRAGMA_ENABLE_DEPRECATION_WARNINGS
```

## CONSTRUCTORS

- `MyProject_Source_MyProject_Public_MyPlayerState_h_16_STANDARD_CONSTRUCTORS`
    - `NO_API AMyPlayerState(const FObjectInitializer& ObjectInitializer)`;
    - `DEFINE_DEFAULT_OBJECT_INITIALIZER_CONSTRUCTOR_CALL(AMyPlayerState)`
    - 带参数 `const FObjectInitializer& ObjectInitializer` 的构造函数
- `MyProject_Source_MyProject_Public_MyPlayerState_h_16_ENHANCED_CONSTRUCTORS`
    - `DEFINE_DEFAULT_CONSTRUCTOR_CALL(AMyPlayerState)`
    - 默认无参构造函数

```c++
#define MyProject_Source_MyProject_Public_MyPlayerState_h_16_STANDARD_CONSTRUCTORS \
	/** Standard constructor, called after all reflected properties have been initialized */ \
	NO_API AMyPlayerState(const FObjectInitializer& ObjectInitializer); \
	DEFINE_DEFAULT_OBJECT_INITIALIZER_CONSTRUCTOR_CALL(AMyPlayerState) \
	DECLARE_VTABLE_PTR_HELPER_CTOR(NO_API, AMyPlayerState); \
DEFINE_VTABLE_PTR_HELPER_CTOR_CALLER(AMyPlayerState); \
private: \
	/** Private move- and copy-constructors, should never be used */ \
	NO_API AMyPlayerState(AMyPlayerState&&); \
	NO_API AMyPlayerState(const AMyPlayerState&); \
public:


#define MyProject_Source_MyProject_Public_MyPlayerState_h_16_ENHANCED_CONSTRUCTORS \
private: \
	/** Private move- and copy-constructors, should never be used */ \
	NO_API AMyPlayerState(AMyPlayerState&&); \
	NO_API AMyPlayerState(const AMyPlayerState&); \
public: \
	DECLARE_VTABLE_PTR_HELPER_CTOR(NO_API, AMyPlayerState); \
DEFINE_VTABLE_PTR_HELPER_CTOR_CALLER(AMyPlayerState); \
	DEFINE_DEFAULT_CONSTRUCTOR_CALL(AMyPlayerState)
```

## DEFINE_DEFAULT_CONSTRUCTOR_CALL 和 DEFINE_DEFAULT_OBJECT_INITIALIZER_CONSTRUCTOR_CALL

- `DEFINE_DEFAULT_CONSTRUCTOR_CALL`
    - 无参构造函数构造器
- `DEFINE_DEFAULT_OBJECT_INITIALIZER_CONSTRUCTOR_CALL`
    - 带参数 `const FObjectInitializer& X `的构造函数构造器

```c++
//ObjectMacros.h
#define DEFINE_DEFAULT_CONSTRUCTOR_CALL(TClass) \
	static void __DefaultConstructor(const FObjectInitializer& X) { new((EInternal*)X.GetObj())TClass; }

#define DEFINE_DEFAULT_OBJECT_INITIALIZER_CONSTRUCTOR_CALL(TClass) \
	static void __DefaultConstructor(const FObjectInitializer& X) { new((EInternal*)X.GetObj())TClass(X); }
```