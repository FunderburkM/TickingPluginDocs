# Mono Tick v1.5  

Mono Tick, improved!  

[Marketplace Link](https://www.unrealengine.com/marketplace/en-US/product/mono-ticking-plugin)  

The Concept of Mono Tick is simple. The Engine's ticking process is very capable, and in its capability lies a decent amount of calculations such as things like Ticking Prerequisites, especial ticking groups, inidividual vs global time dilation, individual ticking rate calculations, event dispatching and solving, etc. Sometimes, we don't need all of these features, and/or we want to handle things a little differently. This is where Mono Tick helps. You can create your own ticking pools, organize and specify ticking rules, and even batch reduce ticking rates.  

## Important 

On Projects with a very large number of ticking objects, this can result in significant performance gains. Keep in mind that your mileage may vary, and results will depend on the nature of the ticking objects, native (C++) vs blueprint ticking, frequencies, build targets, amongst others.  

Here's a rough example of the difference you can obtain:  

For reference, at 60fps, 1ms = 7-8 fps. At 120fps, 1ms = 17-19 fps. 

  
**Setup 1**: 2000 registered tick objects to Mono Tick instead of Engine Tick  

> Tested on Computer with a 7950x and 128gb of ddr5 6000hz ram  

Average Differences:  

* PIE: 3.4ms [Mono Tick] vs 4.9ms [Engine Tick] - Delta: 1.5ms  
* Standalone: 2.1ms [Mono Tick] vs 3.2ms [Engine Tick] - Delta: 1.1ms  
* Development Packaged: 1.5ms [Mono Tick] vs 2.3ms [Engine Tick] - Delta: 0.8ms  
* Shipping Packaged: 1.2ms [Mono Tick] vs 1.8ms [Engine Tick] - Delta: 0.6ms  

**Setup 2**: 2000 registered tick objects to Mono Tick instead of Engine Tick  

> Tested on Computer with a 3700x and 48gb of ddr4 3000hz ram  

Average Differences:  

* PIE: 4.2ms [Mono Tick] vs 5.9ms [Engine Tick] - Delta: 1.7ms  
* Standalone: 3.6ms [Mono Tick] vs 4.9ms [Engine Tick] - Delta: 1.3ms  
* Development Packaged: 3.0ms [Mono Tick] vs 4.0ms [Engine Tick] - Delta: 1.0ms  
* Shipping Packaged: 2.6ms [Mono Tick] vs 3.5ms [Engine Tick] - Delta: 0.9ms  

**Setup 3**: 25000 registered tick objects on Mono Tick instead of Engine Tick  
> Tested on Computer with a 7950x and 128gb of ddr5 6000hz ram  

* PIE: 39.3ms [Mono Tick] vs 69.7ms [Engine Tick] - Delta: 30ms  
* Standalone: 14.6ms [Mono Tick] vs 43.5ms [Engine Tick] - Delta: 28.9ms  
* Development Packaged: 13.0ms [Mono Tick] vs 40.2ms [Engine Tick] - Delta: 27.2 ms  
* Shipping Packaged: 10.5ms [Mono Tick] vs 36.2ms [Engine Tick] - Delta: 25.7ms  




## Table of Contents

- [Update Notes](#update-important-aspects)  
- [Enabling the Plugin](#enabling-the-plugin)  
- [Locating Plugin Files](#locating-plugin-sample-files)  
- [Plugin Project Settings](#project-settings)  
- [Ticking Groups](#ticking-groups)  
- [Subsystem Calls](#subsystem-calls)   

## Pages to Explore 

- [Implementation](/Implementation.md) - For how to implement the ticking system in both C++ and Blueprints  
- [How To](/HowTo.md) - For Example cases when working with the ticking system.  


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

Whether you need to choose Show Engine Content or Show Plugin Content depends on:  

* Show Plugin Content if the plugin is in `/YourProject/Plugins/CM_Tick_Module`  
* Show Engine Content otherwise if in `/TheEngine/Plugins/Marketplace/CM_Tick_Module`.  


![Image](/Resources/MT_ContentBrowser.png)  

You can check the sample testing content here. It spawns 2000 instances of the default testing actor and ticks them using mono tick instead of the engine's default ticking process:  

![Image](/Resources/MT_Sample_SpawnerSettings.png)  

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