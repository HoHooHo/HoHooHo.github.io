# Attribute
----------------------

## Attribute

`Attribute` 是由 `FGameplayAttributeData` `定义的浮点值。通常用来表示角色的某些属性，如：HP、MP等等。Attribute` 通常只被 `GameplayEffect` 修改，因此 `ASC` 可以预测这个修改。

`Attribute` 被定义在 `AttributeSet` `中，AttributeSet` 也会负责处理 `Attribute` 的网络复制。


!!! tip "HideInDetailsView"
    如果不需要 `Attribute` 显示在编辑器的属性详情中，可以使用` meta = (HideInDetailsView)` 属性说明符。

## BaseValue 和 CurrentValue

一个 `Attribute` 由两个值构成，`BaseValue` 和 `CurrentValue`。

  - BaseValue：Attribute 的永久基值
  - CurrentValue：Attribute 的 当前值，其 BaseValue + GameEffects 的临时修改值


!!! tip "应用 `GameplayEffect` 时修改 `Attribute` 的值"
    - `Instant`：修改 `BaseValue`
    - `Duration`：修改 `CurrentValue`
        - 当启用 `Periodic` 时，修改 `BaseValue`
    - `Infinite`：修改 `CurrentValue`
        - 当启用 `Periodic` 时，修改 `BaseValue`


## 监听 Attribute 的变化

```c++
//设置代理
AbilitySystemComponent->GetGameplayAttributeValueChangeDelegate(AttributeSet->GetHealthAttribute()).AddUObject(this, &ARPGPlayerState::HealthChanged);

//回调函数
//FGameplayEffectModCallbackData 只能在服务端上设置
virtual void HealthChanged(const FOnAttributeChangeData& Data)；
```

