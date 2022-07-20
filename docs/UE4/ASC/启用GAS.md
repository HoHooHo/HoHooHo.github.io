# 启用GAS
----------------------

使用GAS的基本步骤：

  - 在虚幻引擎编辑器中启用 `GameplayAbilitySystem` 插件
  - 在[ProjectName].Build.cs文件中的 `PrivateDependencyModuleNames`，添加依赖模块
      - GameplayTags
      - GameplayAbilities
      - GameplayTasks

```c++
// Copyright Epic Games, Inc. All Rights Reserved.

using UnrealBuildTool;

public class GASTest : ModuleRules
{
	public GASTest(ReadOnlyTargetRules Target) : base(Target)
	{
		PCHUsage = PCHUsageMode.UseExplicitOrSharedPCHs;

		PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", "HeadMountedDisplay" });
		PrivateDependencyModuleNames.AddRange(new string[] { "GameplayAbilities", "GameplayTags", "GameplayTasks" });
	}
}
```

!!! warning "UAbilitySystemGlobals::InitGlobalData"
    从4.24开始，必须要调用 UAbilitySystemGlobals::Get().InitGlobalData() 才能使用 TargetData。