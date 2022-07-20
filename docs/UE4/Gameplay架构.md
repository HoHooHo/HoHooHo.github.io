# Server-Client架构
----------------------


## 架构
| ServerOnly | Server&Clients | Server&OwningClient | OwningClientOnly |
| :-----| :---- | :---- | :---- |
| AGameMode | AGameState & APlayerState & APawn | APlayerController| AHUD & UMG |


![ue4-server-client](./assets/ue4-server-client.png)


## AGameMode

```c++
namespace MatchState
{
	const FName EnteringMap      = FName(TEXT("EnteringMap"));
	const FName WaitingToStart   = FName(TEXT("WaitingToStart"));
	const FName InProgress       = FName(TEXT("InProgress"));
	const FName WaitingPostMatch = FName(TEXT("WaitingPostMatch"));
	const FName LeavingMap       = FName(TEXT("LeavingMap"));
	const FName Aborted          = FName(TEXT("Aborted"));
}
```


## RPC

!!! tip "Server、Client、NetMulticast"
    声明 RPC 函数时，会声明一个名为 `函数名_Implementation` 的附加函数，而这个附加函数才是我们需要实现的 远程函数


!!! tip "Reliable、Unreliable"
    由于带宽或网络错误会导致 RPC 失败，当声明为 Reliable 时，会确保 RPC 成功


!!! tip "WithValidation"
    声明 RPC 函数时，会声明一个名为 `函数名_Validate` 的附加函数，此函数使用相同的参数，但是会返回`bool值`，只有返回 `true` 时才会成功调用 RPC


![ue4_rpc_authority](./assets/ue4_rpc_authority.png)
