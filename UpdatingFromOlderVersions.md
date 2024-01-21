# Updating from Older Versions

## Overview

A lot of the previous flow with working with Mono Tick v1 and v1.1 has changed. Upon updating an existing project, you may get several compiling errors on your implementations. Here are the general notion of what's changed:  

* The previous six ticking groups by priority don't exist anymore in the same way they used to. 
* You don't need to use `CM_Tick_GameMode` nor `CM_Tick_GameState` anymore. Those classes don't do anything anymore.
* You don't need to use the subsystem call `Tick` (Also known as `Tick All Groups`). This function has been deprecated and now the ticking subsystem handles all automatically handled groups (check below).
* The necessary interface implementation has been significantly thinned out.  

## Bug Fixes

* There was a bug sometimes where removing many objects in a single frame would result some objects skipping ticking. That has been solved.  

## Deprecation

### Game Mode + Game State

The classes `CM_Tick_GameMode` and `CM_Tick_GameState` (and its Blueprint equivalents in the test content) are deprecated and no longer do anything. Feel free to remove that inheritance from either the game mode or game state in your projects. They are now effectively empty.  

### Ticking Interface 

EVERY Function in the Ticking interface `ICI_WAdmin_Tick` but I_Tick_Run has been marked as deprecated and should not be used anywhere. This means that if your project is still calling things like InitialAdd (whether in blueprint or in `ICI_WAdmin_Tick::Execute_InitialAdd(this)`), please remove it from your project.  You can check [Implementation](/Implementation.md) for more information on this.  

To reiterate, the following functions have been marked as deprecated:

	* SetTick()  
    * SetCanTick()  
    * InitialAdd()  
    * I_Tick_GetOwner()  
    
This applies for both the blueprint and the C++ _Implementation() overrides. Please **REMOVE** any implementations/overrides and calls to these functions in your projects.  

## Ticking Groups

Previously, the 6 groups ticked in order via the global tick() function call. Now, it is managed by the project settings, where the user can specify how the ticking groups behave. The Order of group declaration and addition still matters. Any two groups that tick in the same frame will still depend on addition order, so if you do not change the default settings in terms of tick rate, the behavior will remain the same.  

Automatic ticking has been added to the plugin! Calling the Tick All / tick function for global matters is no longer necessary. You can still call ticking on groups manually for any group that you tick manually and/or that you want to force ticking at a given time during a frame (if it hasn't already).  