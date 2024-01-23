# Mono Tick v1.5  

Mono Tick v1.5 is available for UE5.1 and above. Earlier plugin engine versions will be on 1.1.

Mono Tick, improved!  

[Marketplace Link](https://www.unrealengine.com/marketplace/en-US/product/mono-ticking-plugin)  

The Engine's ticking process is very capable, and in its capability lies a decent amount of calculations such as things like Ticking Prerequisites, especial ticking groups, individual vs global time dilation, individual ticking rate calculations, event dispatching and solving, etc. Often times, we don't need all of this feature processing, we want to bypass them, and/or we want to handle things a little differently.  

This is where Mono Tick shines. Create your own ticking pools, organize and specify ticking rules, and even batch reduce ticking rates. Abstract the ticking process one level above the object, improving cache access and cpu cycling.    

## Important 

- **IMPORTANT** [Updating from older versions](/UpdatingFromOlderVersions.md)  
- [Performance and Good to Check Notes](/Notes.md)  
- [Hotfixes and sub release notes](/HotFixUpdates.md)  

## Table of Contents

- [Enabling the Plugin](#enabling-the-plugin)  
- [Locating Plugin Files](#locating-plugin-sample-files)  
- [Plugin Project Settings](#project-settings)  
- [Ticking Groups](#ticking-groups)  
- [Subsystem Calls](#subsystem-calls)   

## Pages to Explore 

- [Implementation](/Implementation.md) - For how to implement the ticking system in both C++ and Blueprints  
- [How To](/HowTo.md) - For Example cases when working with the ticking system.  


----

## Enabling The Plugin

Go to the Plugins Tab, and search for the plugin. You will either see it as `CM_Tick_Module` or `Mono Tick`. Enable the plugin, and restart the project.  

![Image](/Resources/Plugins_Enable_CM.png)  
![Image](/Resources/Plugins_Enable_MT.png)  

The Plugin's Friendly name is `Mono Tick`, but its technical name in source code is `CM_Tick_Module`. The name has maintained due to legacy reasons.  

## Project Settings

Mono Tick v1.5 comes with its dedicated Project Setting Section. To Locate it, go to Project Settings Tag, and search for the plugins section. In it, locate `Mono Tick Settings`.  

![Image](/Resources/MT_ProjectSettings.png)  

This section lets you specify global ticking settings, as well as set up any defaults, including the pre-registered ticking groups available. Note: Check [How to](/HowTo.md) for information on tick group registration at runtime.  

![Image](/Resources/MT_ProjectSettings_2.png)  


### Ticking Groups

A Ticking Group has the following options:  

- A Developer Name. Used for Debugging and `By Name` Query Functions (check subsystem calls).  
- Ticking Settings, such as enable toggle.  
- Ticking Settings, such as whether the group can tick while the game world is in paused state.  
- Ticking Automatic Settings. This determines whether our system automatically manages this ticking group or whether the project developer manually ticks this group (check subsystem calls).  
- Ticks on Server or Client. whether the client or server are allowed this group to be ticked.  
- Tick Rate - How often should this group tick whenever it is an automatically managed group. 0 means tick every frame.  

Check [How To's Page](/Implementation.md) for information on the different actions you can take, such as creating and removing groups, modifying tick rate, toggles, and manually or automatically ticking! 

## Subsystem Calls

The subsystems calls happen mostly via the function library `UCL_WAdmin`.  

![Image](/Resources/MT_SubsystemCalls.png)  

## Locating Plugin Sample Files  

**The plugin comes with sample content that you can check and test for. This Sample content is not required to be used for the plugin to work but rather serves as a point of reference**.  

Go to Content Folder -> All. It should appear as either `CM_Tick_Module Content` or `Mono Tick Content`. If it doesn't show up after enabling the plugin and restarting the project, check that the correct show option is enabled in the content browser settings panel.  

Whether you need to choose Show Engine Content or Show Plugin Content depends on:  

* Show Plugin Content if the plugin is in `/YourProject/Plugins/CM_Tick_Module`  
* Show Engine Content otherwise if in `/TheEngine/Plugins/Marketplace/CM_Tick_Module`.  


![Image](/Resources/MT_ContentBrowser.png)  

You can check the sample testing content here. It spawns 2000 instances of the default testing actor and ticks them using mono tick instead of the engine's default ticking process:  

![Image](/Resources/MT_Sample_SpawnerSettings.png)  
