# Notes  

## More about the Plugin

The Plugin is using a world [subsystem](https://docs.unrealengine.com/en-US/Programming/Subsystems/index.html) to operate. Specifically, a TickableWorldSubsystem and is present on any game type of world, whether that is play in editor or in game (standalone, packaged, etc).  

The Plugin's inpiration lies in one of GDC's talks featuring Sea of Thieves.  

This plugin can circumvent regular object ticking for actors and components as well as provide ticking support to any object _as long as they access to a world_.  

What does this mean?  

Every reflected object class (such as UObject, UActorComponent, AActor, etc) has a function called  

```cpp
virtual UWorld* GetWorld() const;  
```

This function is used to get a world context. [What is a world](https://docs.unrealengine.com/4.27/en-US/API/Runtime/Engine/Engine/UWorld/)? By default, it tries to get the world from its outer. You may see how when you create an object in blueprint there's an exposed variable called `Outer`, and in C++ doing intakes an outer in the form of `NewObject<UYourType>(OuterObject)`. You can however override this in C++ and have it query from anything you may need if the outer isn't a viable way to reaching its world. Keep in mind that the default implementation of `GetWorld()` is recursive on its Outer, so if your immediate Outer can't get a world, _it_ will try to get the woorld from its Outer, and so forth.  

### Tip

This is what controls whether you see the function input `World Context Object` on many functions such as tracing, global getters, etc. The engine's blueprint machine checks if an object overrides the function and ensures that it is not called on the default implementation whenever the object is a template (refer to CDOs/Template objects in C++ terms). To circumvent this, all you need to do is  

```cpp
UWorld* UMyObject::GetWorld() const
{
    if (IsTemplate())
    {
        return nullptr;
    }
    //Whatever gets the world, even calling Super::GetWorld() works here. 
    return GetOuter()->GetWorld();
}
```


## Performance Implications  

On Projects with a very large number of ticking objects, this can result in significant performance gains. Keep in mind that your mileage may vary, and results will depend on the nature of the ticking objects, native (C++) vs blueprint ticking, frequencies, build targets, amongst others.  

Here's a rough example of the difference you can obtain:  

For reference, at 60fps, 1ms = 7-8 fps. At 120fps, 1ms = 17-19 fps. 

  
**Setup 1**: 2000 Blueprint-derived registered tick objects to Mono Tick instead of Engine Tick  

> Tested on Computer with a 7950x and 128gb of ddr5 6000hz ram  

Average Differences:  

* PIE: 3.4ms [Mono Tick] vs 4.9ms [Engine Tick] - Delta: 1.5ms  
* Standalone: 2.1ms [Mono Tick] vs 3.2ms [Engine Tick] - Delta: 1.1ms  
* Development Packaged: 1.5ms [Mono Tick] vs 2.3ms [Engine Tick] - Delta: 0.8ms  
* Shipping Packaged: 1.2ms [Mono Tick] vs 1.8ms [Engine Tick] - Delta: 0.6ms  

**Setup 2**: 2000 Blueprint-derived registered tick objects to Mono Tick instead of Engine Tick  

> Tested on Computer with a 3700x and 48gb of ddr4 3000hz ram  

Average Differences:  

* PIE: 4.2ms [Mono Tick] vs 5.9ms [Engine Tick] - Delta: 1.7ms  
* Standalone: 3.6ms [Mono Tick] vs 4.9ms [Engine Tick] - Delta: 1.3ms  
* Development Packaged: 3.0ms [Mono Tick] vs 4.0ms [Engine Tick] - Delta: 1.0ms  
* Shipping Packaged: 2.6ms [Mono Tick] vs 3.5ms [Engine Tick] - Delta: 0.9ms  

**Setup 3**: 25000 Blueprint-derived registered tick objects on Mono Tick instead of Engine Tick  
> Tested on Computer with a 7950x and 128gb of ddr5 6000hz ram  

* PIE: 39.3ms [Mono Tick] vs 69.7ms [Engine Tick] - Delta: 30ms  
* Standalone: 14.6ms [Mono Tick] vs 43.5ms [Engine Tick] - Delta: 28.9ms  
* Development Packaged: 13.0ms [Mono Tick] vs 40.2ms [Engine Tick] - Delta: 27.2 ms  
* Shipping Packaged: 10.5ms [Mono Tick] vs 36.2ms [Engine Tick] - Delta: 25.7ms  
