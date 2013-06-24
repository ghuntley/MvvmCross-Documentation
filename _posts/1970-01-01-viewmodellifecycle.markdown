---
layout: docs
categories: docs
title:  "ViewModel Lifecycle"
---

#ViewModelLifecycle
In MvvmCross v3 - Hot Tuna - the default ViewModel location and construction was overhauled in order to provide 3 new features:

- constructor based Dependency Injection
- navigation using Typed navigation classes
- saving and reloading VM state for 'tombstoning'

These changes were breaking changes for existing v1 and vNext apps, but provide significant testability and usability advantages for MvvmCross developers.

## How ViewModels are Created in v3

The default ViewModelLocator in v3 builds new ViewModel instances using a 4-step process - CIRS:

1. `Construction` - using IoC for Dependency Injection
2. `Init()` - initialisation of navigation parameters
3. `ReloadState()` - rehydration after tombstoning
4. `Start()` - called when initialisation and rehydration are complete


###1 Construction

In MvvmCross, you can navigate to a `ViewModel` using parameter like:

    ShowViewModel<DetailViewModel>( 
      new 
      {
        First="Hello",
        Second="World",
        Answer=42
      });


In older version of MvvmCross, these navigation parameters were passed to the constructor of the `ViewModel`.

However, from v3 moving forwards, these navigation parameters are instead passed to the `Init()` method, and the constructor is now free to be used for Dependency Injection.

This means that, for example, a `DetailViewModel` constructor might now look like:

    public class DetailViewModel : MvxViewModel
    {
      private readonly IDetailRepository _repository;
 
      public DetailViewModel(IDetailRepository repository)
      {
        _repository = repository;
      }
 
      // ...
    }

This Dependency Injection is, of course, optional - you code can instead continue to use ServiceLocation if you prefer:


    public class DetailViewModel : MvxViewModel
    {
      private readonly IDetailRepository _repository;
 
      public DetailViewModel()
      {
        repository = Mvx.Resolve<IDetailRepository>();
      }
 
      // ...
    }


###2. Init()

Now that the construction is used for Dependency Injection, the navigation parameters move to a new method - `Init()`

`Init()` will always be called after construction and before `ReloadState()` and `Start()`

`Init()` can come in several flavors:.

- individual simply-Typed parameters
- a single Typed parameter object with simply-Typed properties
- as `InitFromBundle()` with an `IMvxBundle` parameter - this last flavor is always supported via the `IMvxViewModel` interface.

You can declare zero or more of each of these types, but generally you will probably only want to use one within your application.

So, for example, to support the navigation:

    RequestNavigate<DetailViewModel>(new { First="Hello", Second="World", Answer=42 });

you could implement any of:

    public class DetailViewModel : MvxViewModel
    {
      // ...
    
      public void Init(string First, string Second, int Answer)
      {
        // use the values
      }

      // ...
    }

or:

    public class DetailViewModel : MvxViewModel
    {
      // ...
    
      public class NavObject
      {
        public string First {get;set;}
        public string Second {get;set;}
        public int Answer {get;set;}
      }
    
      public void Init(NavObject navObject)
      {
      // use navObject
      }
    
      // ...
    }

or:

    public class DetailViewModel : MvxViewModel
    {  
      // ...
    
      public override void InitFromBundle(IMvxBundle bundle)
      {
        // use bundle - e.g. bundle.Data["First"]
      }
    
      // ...
    }


Note that multiple calls can be used together if required. This allows for some separation of logic in your code:

    public class DetailViewModel : MvxViewModel
    {
      // ...
    
      public class FirstNavObject
      {
        public string First {get;set;}
        public string Second {get;set;}
      }
 
      public class SecondNavObject
      {
        public int Answer {get;set;}
      }
 
      public void Init(FirstNavObject firstNavObject)
      {
        // use firstNavObject
      }
 
      public void Init(SecondNavObject secondNavObject)
      {
        // use secondNavObject
      }
 
      // ...
    }


##3. ReloadState

If the `View`/`ViewModel` is recovering from a Tombstoned state, then `ReloadState` will be called with the data needed for rehydration.

If there is no saved state then no `ReloadState()` methods will be called. 

Exactly as with `Init()`, `ReloadState` can be called in several different ways.

- individual simply-Typed parameters
- a single Typed parameter object with simply-Typed properties
- as `ReloadStateFromBundle()` using an `IMvxBundle` parameter - this last flavor is always supported via the `IMvxViewModel` interface.

Normally, I'd expect this to be called as:

    public class DetailViewModel : MvxViewModel
    {
      // ...
    
      public class SavedState
      {
        public string Name {get;set;}
        public int Position {get;set;}
      }
    
      public void ReloadState(SavedState savedState)
      {
        // use savedState
      }
 
      // ...
    }


###Aside: where does the SavedState come from?

One of the new ViewModel APIs available in Hot Tuna is a SaveState pattern.

This can be implemented in one of two ways:

- using one or more paremeterless methods that return Typed state objects
- using the override `SavedStateToBundle(IMvxBundle bundle)`

Using a Typed state object:

    public class DetailViewModel : MvxViewModel
    {
      // ...
    
      public class SavedState
      {
        public string Name {get;set;}
        public int Position {get;set;}
      }
 
      public SavedState SaveState()
      {
        return new SavedState()
        {
          Name = _name,
          Position = _position
        };
      }
 
      // ...
    }

Using `SavedStateToBundle`:

    public class DetailViewModel : MvxViewModel
    {
      // ...
 
      protected override void SaveStateToBundle(IMvxBundle bundle)
      {
        bundle.Data["Name"] = _name;
        bundle.Data["Position"] = _position.ToString();
      }
    
      // ...
    }


##4. Start()

After all of `Construction`, `Init`, and `ReloadState` is complete, then the `Start()` method will be called.

This method is simply:

    public class DetailViewModel : MvxViewModel
    {
      // ...
    
      public override void Start()
      {
        // do any start
      }
 
      // ...
    }


## Putting it all together

For a real app, I would expect the navigation, construction and state saving/loading code to actually look like:

    ShowViewModel<DetailViewModel>(
      new DetailViewMode.NavObject
      {
        First = "Hello",
        Second = "World",
        Answer = 42
      });


and

    public class DetailViewModel : MvxViewModel
    {
      public class SavedState
      {
        public string Name {get;set;}
        public int Position {get;set;}
      }
    
      public class NavObject
      {
        public string First {get;set;}
        public string Second {get;set;}
        public int Answer {get;set;}
      }
 
      private readonly IDetailRepository _repository;
    
      public DetailViewModel(IDetailRepository repository)
      {
        _repository = repository;
      }

      public void Init(NavObject navObject)
      {
        // use navObject
      }
    
      public void ReloadState(SavedState savedState)
      {
        // use savedState
      }
    
      public override void Start()
      {
        // do any start
      }
    
      public SavedState SaveState()
      {
        return new SavedState()
        {
          Name = _name,
          Position = _position
        };
      }
    
      // ...
    }


##Overriding CIRS.

If you don't like this `CIRS` (Construction-Init-ReloadState-Start) flow for building your ViewModels, then the good news is that you can easily override the `ViewModelLocator` within v3, just as you could within earlier MvvmCross versions. For more on this see LINK-TODO
