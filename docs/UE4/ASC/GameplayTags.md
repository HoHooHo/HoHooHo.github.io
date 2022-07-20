# GameplayTags
----------------------

## FGameplayTag

FGameplayTag 是一系列层次化的名字，如 `Parent.Child.Grandchild`

  - `GameplayTag` 通过 `GameplayTagManager` 进行注册
  - `UAbilitySystemComponent` 实现了 `IGameplayTagAssetInterface` 接口中的方法以便访问它拥有的 `GameplayTags`


!!! tip "在C++中获取GameplayTag对象"
    FGameplayTag::RequestGameplayTag(FName("Your.GameplayTag.Name"))

## FGameplayTagContainer

  - 多个 `FGameplayTag` 可以被存储到 `FGameplayTagContainer` 中
  - `FGameplayTag` 存储在 `FGameplayTagCountContainer` 中，有一个 `TagMap`，存储了 `GameplayTag` 实例的数量
  - 任何 `HasTag()` 或 `HasMatchingTag()` 或其他类似的方法都会检查 `TagMapCount`，如果 `GameplayTag` 不存在或者 `TagMapCount = 0` 将返回 `false`

!!! tip "`GameplayTagContainer` 与 `TArray<FGameplayTag>`"
    - 强烈建议使用 `GameplayTagContainer` 而不是 `TArray<FGameplayTag>`，因为 `GameplayTagContainers` 有一些极其高效的函数
    - `FGameplayTag` 是标准的 `FName`，在 `FGameplayTagContainers` 中 `FGameplayTag` 可以被高效的打包在一起以完成网络复制
        - 需要先在项目设置中 开启`Fast Replication`。`Fast Replication` 要求服务器和客户端拥有相同的 `GameplayTags` 列表
    - 为了遍历， `GameplayTagContainers` 也可以返回一个 `TArray<FGameplayTag>`。


## GameplayTagManager

对于获取 `GameplayTag` 的父或子标签这类处理，可以通过 `GameplayTagManager` 中的一系列函数完成。 `GameplayTagManager` 实际上以关系节点的方式存储了 `GameplayTags`， 速度上要远优于字符串的处理和比较。


## GameplayTag 编辑器

`GameplayTag` 需要在 `DefaultGameplayTags.ini` 中定义。 UE4 的编辑器在项目设置中提供了一个界面可以让开发者管理 `GameplayTags` 而不需要手动编辑 `DefaultGameplayTags.ini`。`GameplayTag` 编辑器可以创建、重命名、删除 `GameplayTag`，也可以查找 `GameplayTag` 的引用。


!!! tip "查找 `GameplayTag` 的引用"
    查找 `GameplayTag` 的引用只会查找蓝图中的引用。 

!!! tip "`GameplayTag` 的重命名" 
    - 重命名 `GameplayTag` 将会创建一个重定向，相关资源仍然引用原来的 `GameplayTag`，原来的`GameplayTag` 将会被定向到新的 `GameplayTag`。
    - 推荐创建一个新的 `GameplayTag`, 然后手动更新所有引用到这个新`GameplayTag`，然后删除旧的`GameplayTag`，这样可以避免重定向。

    
另外，当开启`Fast Replication`后，`GameplayTag`编辑器可配置进一步优化 `GameplayTags` 的网络复制。

## LooseGameplayTags

由 `GameplayEffect` 添加的 `GameplayTag` 会被复制。 `ASC` 也可以添加不会被复制并且需要手动管理的 `LooseGameplayTags`

!!! tip "LooseGameplayTag"
    可以使用 UAbilitySystemComponent::AddLooseGameplayTag() 和 UAbilitySystemComponent::RemoveLooseGameplayTag()

## GameplayTag 过滤

  - `UPROPERTY(Meta = (Categories = "XX"))`
    - `GameplayTags` 和 `GameplayTagContainers` 属性有可选的 `UPROPERTY` 说明符，可以用来在蓝图中实现标签的过滤，仅展示出父标签为 `XX` 的 `GameplayTags`。
  - `UFUNCTION(Meta = (GameplayTagFilter = "XX"))`
    - 当把 `GameplayTag` 当作函数的参数时，可以通过 `Meta = (GameplayTagFilter = XX) `完成过滤。
    - `GameplayTagContainer` 参数不能过滤

## 监听 FGameplayTag 的变化

`ASC` 可以使用枚举 `EGameplayTagEventType` 来注册代理监听 `GameplayTag` 的变化。

  - EGameplayTagEventType
    - NewOrRemoved ： Event only happens when tag is new or completely removed
    - AnyCountChange ： Event happens any time tag “count” changes

```c++
AbilitySystemComponent->RegisterGameplayTagEvent(FGameplayTag::RequestGameplayTag(FName("State.Debuff.Stun")), EGameplayTagEventType::NewOrRemoved).AddUObject(this, &AGDPlayerState::StunTagChanged);
 
virtual void StunTagChanged(const FGameplayTag CallbackTag, int32 NewCount);
```