# Mono Tick v1.5  

Mono Tick, improved!  

[Marketplace Link](https://www.unrealengine.com/marketplace/en-US/product/mono-ticking-plugin)  

The Concept of Mono Tick is simple. The Engine's ticking process is very capable, and in its capability lies a decent amount of calculations such as things like Ticking Prerequisites, especial ticking groups, inidividual vs global time dilation, individual ticking rate calculations, event dispatching and solving, etc. Sometimes, we don't need all of these features, and/or we want to handle things a little differently. This is where Mono Tick helps. You can create your own ticking pools, organize and specify ticking rules, and even batch reduce ticking rates.  

On Projects with a very large number of ticking objects, this can result in significant performance gains. Keep in mind that your mileage may vary, and results will depend on the nature of the ticking objects, native (C++) vs blueprint ticking, frequencies, build targets, amongst others.  

**Table of Contents** 

- [Update Notes](#update-important-aspects)
- [Enabling the Plugin](#enabling-the-plugin)
- [Locating Plugin Files](#locating-plugin-sample-files)
- [Plugin Project Settings](#project-settings)
- [Subsystem Calls](#subsystem-calls)
- [Ticking Groups](#ticking-groups)
	- [Creating and Removing Groups](#creating-and-removing-ticking-groups)
  	- [Ticking Each Group](#ticking-each-group)
- [How it works](#how-it-works)
	- [C++ Implementation](#how-it-works---cpp-implementation)
	- [BP Implementation](#how-it-works---bp-implementation)
	- [Additional Info](#how-it-works---additional-information)

----

## Update Important Aspects

A lot of the previous flow with working with Mono Tick v1 and v1.1 has changed. Upon updating an existing project, you may get several compiling errors on your implementations. Here are the general notion of what's changed:  

* The previous six ticking groups by priority don't exist anymore in the same way they used to. Previously, the 6 groups ticked in order via the global tick() function call. Now, it is managed by the project settings, where the user can specify how the ticking groups behave. The Order of group declaration and addition still matters. Any two groups that tick in the same frame will still depend on addition order, so if you do not change the default settings in terms of tick rate, the behavior will remain the same.
* There was a bug sometimes where removing many objects in a single frame would result some objects skipping ticking. That has been solved. 
* You don't need to use `CM_Tick_GameMode` nor `CM_Tick_GameState` anymore. Those classes don't do anything anymore.
* You don't need to use the subsystem call `Tick` (Also known as `Tick All Groups`). This function has been deprecated and now the ticking subsystem handles all automatically handled groups (check below).
* The necessary interface implementation has been significantly thinned out. The following functions have been marked as deprecated:
	* `SetTick(), SetCanTick(), InitialAdd(), I_Tick_GetOwner()`. This applies for both the blueprint and the C++ _Implementation() overrides.

----

## Enabling The Plugin

Go to the Plugins Tab, and search for the plugin. You will either see it as `CM_Tick_Module` or `Mono Tick`. Enable the plugin, and restart the project.  

![Image](/Resources/Plugins_Enable_CM.png)  
![Image](/Resources/Plugins_Enable_MT.png)  

The Plugin's Friendly name is `Mono Tick`, but its technical name in source code is `CM_Tick_Module`. The name has maintained due to legacy reasons.  

## Locating Plugin Sample Files  

Go to Content Folder -> All. It should appear as either `CM_Tick_Module Content` or `Mono Tick Content`. If it doesn't show up after enabling the plugin and restarting the project, check that the correct show option is enabled in the content browser settings panel.  

![Image](/Resources/MT_ContentBrowser.png)  

You can check the sample testing content here. It spawns 2000 instances of the default testing actor and ticks them using mono tick instead of the engine's default ticking process:  

![Image](/Resources/MT_Sample_SpawnerSettings.png)  

## Project Settings

Mono Tick v1.5 comes with its dedicated Project Setting Section. To Locate it, go to Project Settings Tag, and search for the plugins section. In it, locate `Mono Tick Settings`.  

![Image](/Resources/MT_ProjectSettings.png)  

This section lets you specify global ticking settings, as well as set up any defaults, including the pre-registered ticking groups available.  

![Image](/Resources/MT_ProjectSettings_2.png)  

## Subsystem Calls

The subsystems calls happen mostly via the function library `UCL_WAdmin`.  

![Image](/Resources/MT_SubsystemCalls.png)  



## Ticking Groups

A Ticking Group has the following options:  

- A Developer Name. Used for Debugging and `By Name` Query Functions (check below).  
- Ticking Settings, such as enable toggle.  
- Ticking Settings, such as whether the group can tick while the game world is in paused state.  
- Ticking Automatic Settings. This determines whether our system automatically manages this ticking group or whether the project developer manually ticks this group (check below).  
- Ticks on Server or Client. whether the client or server are allowed this group to be ticked.  
- Tick Rate - How often should this group tick whenever it is an automatically managed group. 0 means tick every frame.  

### Creating and Removing Ticking Groups  

A Ticking Group can be created via three ways:  

1. Via the Project Settings Ticking Group Settings Variable.  
2. if `AllowAddingGroupsAtRuntime` is true in `Mono Tick Settings`, via the `Subsystem_Tick_Register_TickingGroup` call. 
3. if both `AllowAddingGroupsAtRuntime` and `AllowAddingGroupsAtRuntime_WhenRunningAddObject` are true in `Mono Tick Settings`, via the `Subsystem_Tick_AddObject` call and the object settings did not find an existing valid group.  

A Ticking Group can be removed in the following way:  

1. if `AllowRemovingGroupsAtRuntime` is true in `Mono Tick Settings` via the `Subsystem_Tick_RemoveTickingGroup` call.  


![Image](/Resources/MT_SubsystemCalls.png)  

### Ticking each Group

A Ticking Group can be ticked in two ways:  

1. By our subsystem is the Ticking group ticks automatically  
2. By an external object manually if the group does not tick automatically. The External object must get our WSS_Admin_Tick subsystem and call TickGroup() on the correct group index.  


## How it Works  

The Set up is simple. Implement the `IC_WAdmin_Tick` interface on any object that you wish to tick pooled. Whether it is an actor, a component, a widget, or _any_ Object class.  

![Image](/Resources/MT_InterfaceSetting.png)  

That's All by and large.  

### How it Works - CPP implementation  

If you implement the interface in C++, this is all you need to do:  

```cpp

//Your .h file
#include "Admin/CL_WAdmin.h"

///....

UCLASS()
class AYourClass : public AActor, public ICI_WAdmin_Tick
{
	GENERATED_BODY()
	
public:	
	// Sets default values for this actor's properties
	AYourClass(const FObjectInitializer& ObjectInitializer);

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	
	// Your custom tick function
	virtual bool I_Tick_Run_Implementation(float DeltaTime) override;

// the rest of your class

};

//--------------------------------------------------
//Your .cpp file

// Sets default values
AYourClass::AYourClass(const FObjectInitializer& ObjectInitializer): Super(ObjectInitializer)
{
    //TickingSettings is a FCSTickingObjectSetting variable that comes with the interface. You can set the defaults for the class if you want different values
    //this is optional, whether you want to set a group index or name that's up to you
    TickingSettings.TickingGroupName = "InventoryActors";

    //disabling the regular tick so we don't tick on both systems (sometimes, you may want this if your mono tick group doesn't tick every frame)
    PrimaryActorTick.bCanEverTick = false;

	//the rest of your constructor
}

void ACMTest_ActorBasic::BeginPlay()
{
	Super::BeginPlay();
	
    //this registers it locally to the system
	UCL_WAdmin::Subsystem_Tick_Add(this, this, TickingSettings);

    //the rest of your beginplay
}

bool ACMTest_ActorBasic::I_Tick_Run_Implementation(float DeltaTime)
{
	//Your ticking code here!
	return true;
}

```  

### How it Works - BP Implementation  

If you implement the interface at the blueprint level, All you need to do afterwards is the same:  

![image](/Resources/MT_Sample_Register.png)  
![image](/Resources/MT_Sample_TickFunction.png)  

I_Tick_Run is equivalent to the regular Event Tick provided by the engine.  


### How it Works - Additional Information

If you want to swap betweek ticking groups, use this call:  

![image](/Resources/MT_Sample_Swap.png)  

If you want to toggle the ticking of an individual object, use this call:  

![image](/Resources/MT_Sample_SetTick.png)  
