# Implementation

> Important: Check the [Notes Section](/Notes.md#more-about-the-plugin) for information on what is required from an object perspective to be compatible with the plugin.  

The Setup is simple. Implement the `CI_WAdmin_Tick` interface on any object that you wish to tick pooled. Whether it is an actor, a component, a widget, or _any_ Object class.  

![Image](/Resources/MT_InterfaceSetting.png)  

You use the subsystem calls `Add Object` and `Remove Object` as well as `Set Object Tick` for managing the objects. For registering, you have the `Ticking Object Setting` struct:  

```cpp
/** Whether the object gets registered with ticking enabled.*/
bool bCanTickOnAddition = true;

/** What ticking group do we register to.*/
int32 TickingGroup;

/** When this value != None, we take priority on this over TickingGroup Index. */
FName TickingGroupName
```

## BP Implementation  

If you implement the interface at the blueprint level, All you need to do is this:  

![image](/Resources/MT_Sample_Register.png)  
![image](/Resources/MT_Sample_TickFunction.png)  

I_Tick_Run is equivalent to the regular Event Tick provided by the engine.  

Remember to disable Start With Tick Enabled for Actors and Components in the defaults panel!  


## CPP implementation  

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