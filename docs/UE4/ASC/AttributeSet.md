# AttributeSet
----------------------

## AttributeSet 定义

`AttributeSet` 负责定义并管理 `Attributes`。


!!! tip "AttributeSet"
    - `Actor` 的 `AttributeSet`，会在其 `ASC` 组件的函数 `UAbilitySystemComponent::InitializeComponent` 中自动注册
    - 一个 `ASC` 可以有 `N` 个 `AttributeSet`
        - `AttributeSet` 的内存开销很低。
    - 同一个 `AttributeSet` 在一个 `ASC` 中最多只能有 `1` 个。


## 定义Attributes

```c++
// RPGAttributeSet.h
//建议定义该宏，它将会为属性生成 Getter和Setter 等函数。
#define ATTRIBUTE_ACCESSORS(ClassName, PropertyName) \
    GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \
    GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName) \
    GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName) \
    GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName)

UPROPERTY(BlueprintReadOnly, Category = "HP", ReplicatedUsing = OnRep_HP)
FGameplayAttributeData HP;
ATTRIBUTE_ACCESSORS(UGTAttributeSet, HP);

UFUNCTION()
virtual void OnRep_HP(const FGameplayAttributeData& OldHP);

//////////////////////////////////////////////////////
// RPGAttributeSet.cpp
void UGTAttributeSet::OnRep_HP(const FGameplayAttributeData& OldHP)
{
	GAMEPLAYATTRIBUTE_REPNOTIFY(UGTAttributeSet, HP, OldHP);
}

void UGTAttributeSet::GetLifetimeReplicatedProps(TArray<class FLifetimeProperty>& OutLifetimeProps) const
{
	Super::GetLifetimeReplicatedProps(OutLifetimeProps);
 
	DOREPLIFETIME_CONDITION_NOTIFY(UGTAttributeSet, HP, COND_None, REPNOTIFY_Always);
}
```

!!! tip "Attribute"
    - `REPNOTIFY_Always`：告诉 `OnRep` 函数，在本地值和服务器下发的值即使相同也会触发（为了预测），默认情况下 `OnRep` 不会触发。
    - 如果 `Attribute` 不需要复制（如：Meta Attribute），那么 `OnRep` 和 `GetLifetimeReplicatedProps` 可以跳过。


## AttributeSet 几个重要的函数

!!! tip "PreAttributeChange"
    - 在 `AttributeSet` 的 `CurrentValue` 被改变之前调用，主要对 `NewValue` 进行 `Clamp` 修正
    - 在这里做的任何限制都不会永久性地修改 `ASC` 中的 `Modifier`, 只会修改查询 `Modifier` 的返回值
        - `GameplayEffectExecutionCalculations` 和 `ModifierMagnitudeCalculations` 这种自所有 `Modifier` 重新计算 `CurrentValue` 的函数需要再次执行 `Clamp` 操作

!!! tip "PostGameplayEffectExecute"
    - 仅在 `Instant GE` 对 `Attribute` 的 `BaseValue` 修改之后触发
    - 当 `PostGameplayEffectExecute` 被调用时，对属性的改变已经发生，但还没有复制给客户端，因此在此处进行 `Clamp` 不会执行两次复制，客户端只会收到 `Clamp` 后的结果。

```c++
class GAMEPLAYABILITIES_API UAttributeSet : public UObject
{
public:
	/**
	 *	Called just before modifying the value of an attribute. AttributeSet can make additional modifications here. Return true to continue, or false to throw out the modification.
	 *	Note this is only called during an 'execute'. E.g., a modification to the 'base value' of an attribute. It is not called during an application of a GameplayEffect, such as a 5 ssecond +10 movement speed buff.
	 */	
	virtual bool PreGameplayEffectExecute(struct FGameplayEffectModCallbackData &Data) { return true; }
	
	/**
	 *	Called just before a GameplayEffect is executed to modify the base value of an attribute. No more changes can be made.
	 *	Note this is only called during an 'execute'. E.g., a modification to the 'base value' of an attribute. It is not called during an application of a GameplayEffect, such as a 5 ssecond +10 movement speed buff.
	 */
	virtual void PostGameplayEffectExecute(const struct FGameplayEffectModCallbackData &Data) { }

	/**
	 *	An "On Aggregator Change" type of event could go here, and that could be called when active gameplay effects are added or removed to an attribute aggregator.
	 *	It is difficult to give all the information in these cases though - aggregators can change for many reasons: being added, being removed, being modified, having a modifier change, immunity, stacking rules, etc.
	 */

	/**
	 *	Called just before any modification happens to an attribute. This is lower level than PreAttributeModify/PostAttribute modify.
	 *	There is no additional context provided here since anything can trigger this. Executed effects, duration based effects, effects being removed, immunity being applied, stacking rules changing, etc.
	 *	This function is meant to enforce things like "Health = Clamp(Health, 0, MaxHealth)" and NOT things like "trigger this extra thing if damage is applied, etc".
	 *	
	 *	NewValue is a mutable reference so you are able to clamp the newly applied value as well.
	 */
	virtual void PreAttributeChange(const FGameplayAttribute& Attribute, float& NewValue) { }

	/**
	 *	This is called just before any modification happens to an attribute's base value when an attribute aggregator exists.
	 *	This function should enforce clamping (presuming you wish to clamp the base value along with the final value in PreAttributeChange)
	 *	This function should NOT invoke gameplay related events or callbacks. Do those in PreAttributeChange() which will be called prior to the
	 *	final value of the attribute actually changing.
	 */
	virtual void PreAttributeBaseChange(const FGameplayAttribute& Attribute, float& NewValue) const { }

	/** Callback for when an FAggregator is created for an attribute in this set. Allows custom setup of FAggregator::EvaluationMetaData */
	virtual void OnAttributeAggregatorCreated(const FGameplayAttribute& Attribute, FAggregator* NewAggregator) const { }
};

```