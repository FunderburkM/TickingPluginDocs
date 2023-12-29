# How To


## Creating and Removing Ticking Groups  

A Ticking Group can be created via three ways:  

1. Via the Project Settings Ticking Group Settings Variable.  
2. if `AllowAddingGroupsAtRuntime` is true in `Mono Tick Settings`, via the `Subsystem_Tick_Register_TickingGroup` call. 
3. if both `AllowAddingGroupsAtRuntime` and `AllowAddingGroupsAtRuntime_WhenRunningAddObject` are true in `Mono Tick Settings`, via the `Subsystem_Tick_AddObject` call and the object settings did not find an existing valid group.  

A Ticking Group can be removed in the following way:  

1. if `AllowRemovingGroupsAtRuntime` is true in `Mono Tick Settings` via the `Subsystem_Tick_RemoveTickingGroup` call.  

![Image](/Resources/MT_CreateAndRemoveGroup.png)  

You can also call `UCL_WAdmin::Subsystem_Tick_ResetTickingGroups()` to empty all current groups and reset them to their defaults. 


## Ticking each Group

A Ticking Group can be ticked in two ways:  

1. By our subsystem is the Ticking group ticks automatically. No Actions from the Developer is necessary other than setting up/registering the group with automatic ticking enabled.  
2. By an external object manually if the group does not tick automatically. The External object must get our WSS_Admin_Tick subsystem and call TickGroup() on the correct group index.  

![Image](/Resources/MT_TickGroupManually.png)  

## Get Ticking Group Data

You can run these to query readable only data, such as number of objects ticking, as well as per group information.  

![Image](/Resources/MT_GetTickData.png)  

## Modify Tick Group Data 

You can also modify tick group data at runtime, such as auto ticking, enable toggling and tick rate modification, as well as server/client ticking allowance.  

![Image](/Resources/MT_TickGroup_Functions.png)  

## Set Tick Group Condition

You can dynamically specify whether a group can tick from anywhere. Here is the setup:  

To Bind, Give the function a valid event delegate. Likewise, to unbind, run the function without a valid delegate bound.  

![Image](/Resources/MT_TickCondition_1.png)  

In the bound function, you can do whatever is needed to manage this information.  

![Image](/Resources/MT_TickCondition_2.png)  

## Switching object groups

![Image](/Resources/MT_Sample_Swap.png)  

## Disabling Per Object Tick Enabled  

![Image](/Resources/MT_SetObjectTickEnabled.png)  