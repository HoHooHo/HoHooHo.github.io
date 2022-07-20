# GameplayEffect
----------------------

## UGameplayEffect

`UGameplayEffect` 是 `Ability` 修改其自身和其他 `Attribute` 和 `GameplayTag` 的容器。

!!! tip "GameplayEffect"
    - `GameplayEffect` 只是一个定义单一游戏效果的数据类, 不应该在其中添加额外的逻辑
    - `GameplayEffect` 通过 `Modifier` 和 `Execution` 修改 `Attribute`
    - `GameplayEffect` 可以 添加/执行 `GameplayCue`
        - `Instant GE` 将调用 `GameplayCue` 的 `Execute`
        - `Duration GE` 或 `Infinite GE` 将调用 `GameplayCue` 的 `Add` 和 `Remove`

  - `Instant GE`
      - 修改 `BaseValue`，调用 `GameplayCue` 的 `Execute`
  - `Duration GE` 和 `Infinite GE`
      - 修改 `CurrentValue`，执行 `GameplayCue` 的 `Add` 和 `Remove`
      - 可以设置 `Period`
          - 每隔 `Period` 秒，则修改一次 `Attributes` 的 `BaseValue`，并调用 `GameplayCue` 的 `Execute`
            - 相当于每隔 `Period` 秒，执行一次 `“Instant”` 类型的效果，直到 `GameplayEffect` 移除。

## 监听 GameplayEffect 的 Add和Remove

```c++
//设置监听
AbilitySystemComponent->OnActiveGameplayEffectAddedDelegateToSelf.AddUObject(this, &ARPGCharacterBase::OnGameplayEffectAddedCallback);
AbilitySystemComponent->OnAnyGameplayEffectRemovedDelegate().AddUObject(this, &ARPGCharacterBase::OnGameplayEffectRemovedCallback);
 
//回调函数
virtual void OnGameplayEffectAddedCallback(UAbilitySystemComponent* Target, const FGameplayEffectSpec& SpecApplied, FActiveGameplayEffectHandle ActiveHandle);
virtual void OnGameplayEffectRemovedCallback(const FActiveGameplayEffect& EffectRemoved);

