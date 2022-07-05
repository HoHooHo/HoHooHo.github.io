# 继承 FTickableGameObject

```c++
UCLASS()
class UCoolDownMgr : public UObject, public FTickableGameObject
{
    GENERATED_BODY()
public:
    // Sets default values for this character's properties
    UCoolDownMgr();
    virtual ~UCoolDownMgr();
    // Begin FTickableGameObject Interface.
    virtual void Tick(float DeltaTime) override;
    virtual bool IsTickable() const override;
    virtual TStatId GetStatId() const override;
    // End FTickableGameObject Interface.
};
```


# 创建代理 FTickerDelegate

```c++
TickDelegate = FTickerDelegate::CreateUObject(this, &UShooterGameInstance::Tick);
TickDelegateHandle = FTicker::GetCoreTicker().AddTicker(TickDelegate);
```