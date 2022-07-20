# AbilitySystemComponent
----------------------

## ASC
`UAbilitySystemComponent(ASC)`是整个技能系统的核心，`ASC` 本质上是一个 `UActorComponent`，用于处理技能系统中的所有交互。

 - `ASC` 持有被赋予的`GameplayAbilities`，`FGameplayAbilitySpecContainer ActivatableAbilities`。
 - `ASC` 持有当前已激活的`GameplayEffects`，`FActiveGameplayEffectsContainer ActiveGameplayEffects`。

!!! warning "迭代`ASC`的`ActivatableAbilities.Items`"
    在迭代`ASC`的`ActivatableAbilities.Items`之前，一定要添加 `ABILITYLIST_SCOPE_LOCK()`，在 `ABILITYLIST_SCOPE_LOCK()` 过程中更不要删除 `Ability`。

## OwnerActor 和 AvatarActor

`ASC` 的 `OwnerActor`和`AvatarActor`可以是同一个`Actor`，如 MOBA 中的野怪。也可以是不同的`Actor`，如 MOBA 中玩家控制的 Character，此时 OwnerActor 通常为 `PlayerState`。
    
!!! tip "IAbilitySystemInterface"
    如果`OwnerActor`和`AvatarActor`是不同的 Actor，则两者都需实现`IAbilitySystemInterface`

!!! tip "`PlayerState` 的 `NetUpdateFrequency`"
    - 如果 `ASC` 的 `OwnerActor` 为 `PlayerState`，则需增加 `PlayerState` 的网络更新频率 `NetUpdateFrequency`，
    - `PlayerState` 的默认更新频率非常低，会导致 `Attributes` 和 `GameplayTags` 的同步延迟，
    - 确保启用了 `Adaptive Network Update Frequency`，默认是关闭的

## ASC复制模式

`ASC` 提供了三种不同的复制模式，用于复制 `GameplayEffects`、`GameplayTags` 和 `GameplayCues`。而 `Attributes` 由 `AttributeSet` 复制。

| 复制模式 | 使用场景 | 描述 |
|  ----  | ----  | ----  |
| Full | 单人 | GameplayEffect 会被复制到所有客户端。 |
| Mixed | 多人，玩家控制的Actor | GameplayEffect仅被复制到 Owner Client，GameplayTags和GameplayCus会被复制到所有客户端。 |
| Minimal | 多人，AI控制的Actor | GameplayEffect不会被复制到任何客户端，GameplayTags和GameplayCus会被复制到所有客户端 |


## ASC的创建和初始化

`ASC` 通常在 `OwnerActor` 的构造函数中创建，并设置复制模式，这一步必须在C++中完成。

``` c++
ARPGPlayerState::ARPGPlayerState()
{
    // Create ability system component, and set it to be explicitly replicated
    AbilitySystemComponent = CreateDefaultSubobject<URPGAbilitySystemComponent>(TEXT("AbilitySystemComponent"));
    AbilitySystemComponent->SetIsReplicated(true);
	AbilitySystemComponent->SetReplicationMode(EGameplayEffectReplicationMode::Mixed);
    
	NetUpdateFrequency = 100.0f;
    //.
}
```

!!! tip "ASC的初始化"
    `ASC` 需要在 `OwnerActor` 和 `AvatarActor` 中都进行初始化，而且必须在`服务器`和`客户端`都要完成初始化。

### 当 OwnerActor == AvatarActor 时

对于玩家控制的角色，ASC 的 OwnerActor和AvatarActor 都为 Pawn 时，通常

    -服务器：在 Pawn 的 PossessedBy() 函数中完成 ASC 的初始化；
    -客户端：在 PlayerController 的 AcknowledgePawn() 函数中完成 ASC 的初始化。


```c++

/* ARPGCharacterBase.cpp */
void ARPGCharacterBase::PossessedBy(AController * NewController)
{
    Super::PossessedBy(NewController);
​
    if (AbilitySystemComponent)
    {
        AbilitySystemComponent->InitAbilityActorInfo(this, this);
    }
    //...
}

/* ARPGPlayerControllerBase.cpp*/
void ARPGPlayerControllerBase::AcknowledgePossession(APawn* P)
{
    Super::AcknowledgePossession(P);
​
    ARPGCharacterBase* CharacterBase = Cast<ARPGCharacterBase>(P);
    if (CharacterBase)
    {
        CharacterBase->GetAbilitySystemComponent()->InitAbilityActorInfo(CharacterBase, CharacterBase);
    }
    //...
}
```

### 当 OwnerActor != AvatarActor 时

对于玩家控制的角色，ASC 的 OwnerActor 为 PlayerState，AvatarActor 为 Pawn 时，通常

    - 服务器：在 Pawn 的 PossessedBy() 函数中完成 ASC 的初始化（与上面相同）；
    - 客户端：在 Pawn 的 OnRep_PlayerState() 函数中 完成 ASC 的初始化（这将确保 PlayerState 在客户端已存在）。

```c++

// Server only
void ARPGHeroCharacter::PossessedBy(AController * NewController)
{
    Super::PossessedBy(NewController);
​
    ARPGPlayerState* PS = GetPlayerState<ARPGPlayerState>();
    if (PS)
    {
        // Set the ASC on the Server. Clients do this in OnRep_PlayerState()
        AbilitySystemComponent = Cast<URPGAbilitySystemComponent>(PS->GetAbilitySystemComponent());
​
        // AI won't have PlayerControllers so we can init again here just to be sure. No harm in initing twice for heroes that have PlayerControllers.
        PS->GetAbilitySystemComponent()->InitAbilityActorInfo(PS, this);
    }
    
    //...
}

// Client only
void ARPGHeroCharacter::OnRep_PlayerState()
{
    Super::OnRep_PlayerState();
​
    ARPGPlayerState* PS = GetPlayerState<ARPGPlayerState>();
    if (PS)
    {
        // Set the ASC for clients. Server does this in PossessedBy.
        AbilitySystemComponent = Cast<URPGAbilitySystemComponent>(PS->GetAbilitySystemComponent());
​
        // Init ASC Actor Info for clients. Server will init its ASC when it possesses a new Actor.
        AbilitySystemComponent->InitAbilityActorInfo(PS, this);
    }
​
    // ...
}
```


!!! warning "如果看到如下日志，则说明没有在客户端初始化 ASC"
    LogAbilitySystem: Warning: Can't activate LocalOnly or LocalPredicted ability %s when not local!