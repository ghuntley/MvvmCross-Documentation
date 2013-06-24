---
layout: docs
categories: docs
title:  "Customising Setup"
---

#Customisation opportunities

In each deployed MvvmCross application there are two key classes which control how your app starts:

- the `App` class in the core project - which provides the initialization for your core business logic and your viewmodels.
- the `Setup` class in the native UI project - this `Setup` is a bootstrapper for the MvvmCross system and for your app.

## App.cs

Typically App.cs provides only initialization of:

- simple rule-based IoC registration - e.g.:
 
            CreatableTypes()
                .EndingWith("Service")
                .AsInterfaces()
                .RegisterAsLazySingleton();

- the `ViewModelLocator` - how `ViewModel`s are found or created when `View`s are displayed
- the `IMvxAppStart` - which `ViewModel` or `ViewModel`s are shown when the application is first started

## Setup.cs

Internally the `Setup` bootstrapper performa many steps. 

You can see most of them in the MvxSetup.cs class source which includes a sequence like this:

            // IoC
            InitializeIoC();
         
            // Core components
            InitializeFirstChance();
            InitializeDebugServices();
            InitializePlatformServices();
            InitializeSettings();
            InitializeSingletonCache();
            
            // Second components
            PerformBootstrapActions();
            InitializeStringToTypeParser();
            InitializeViewModelFramework();
            var pluginManager = InitializePluginFramework();
            InitializeApp(pluginManager);
            InitialiseViewModelTypeFinder();
            InitializeViewsContainer();
            InitiaiseViewDispatcher();
            InitializeViewLookup();
            InitialiseCommandCollectionBuilder();
            InitializeNavigationSerializer();
            InitializeInpcInterception();
            InitializeLastChance();

Most of these steps are `virtual` - so they allow customisation. Also most of these steps are implemented using virtual `Create` steps - which again should make customisation easier:
           
            protected virtual void InitialiseFoo()
            {
               var foo = CreateFoo();
               Mvx.RegisterSingleton<Foo>();
            }
            
            protected virtual IFoo CreateFoo()
            {
               return new Foo();
            }

Added to these base class steps, each platform adds a small number of platform specific steps - eg Android adds some additional methods and properties for initialisation of the Android UI and especially of the data-binding framework:

            string ExecutableNamespace { get; }
            Assembly ExecutableAssembly { get; }
            IMvxAndroidViewPresenter CreateViewPresenter()
            InitializeSavedStateConverter()
            InitialiseBindingBuilder()
            MvxAndroidBindingBuilder CreateBindingBuilder()
            RegisterBindingBuilderCallbacks()
            FillBindingNames(IMvxBindingNameRegistry registry)
            FillAxmlViewTypeResolver(IMvxAxmlNameViewTypeResolver viewTypeResolver)
            FillNamespaceListViewTypeResolver(IMvxNamespaceListViewTypeResolver viewTypeResolver)
            FillValueConverters(IMvxValueConverterRegistry registry)
            FillTargetFactories(IMvxTargetBindingFactoryRegistry registry)
            IList<string> ViewNamespaces { get; }
            IDictionary<string, string> ViewNamespaceAbbreviations { get; }
            List<Assembly> ValueConverterAssemblies { get; }
            List<Type> ValueConverterHolders { get; }
            IList<Assembly> AndroidViewAssemblies { get; }

This long list of virtual methods does provide a lot of opportunities for overriding default MvvmCross behaviour. 

However, **most applications actually override only very few of these methods**. Indeed, many applications only override only the one required method: `IMvxApplication CreateApp()`

If you do want to override some of the Setup, then the rest of this document describes some of the methods and properties that you may find useful.

#Individual customisations

##Providing application specific initialisation

There are three key methods where application specific initialisation might be added

1 `App.Initialize` - this is the place where all cross-platform app initialisation should occur. In general it is the **first choice** for all app-specific initialization. Only use the `Setup`-based methods if you need platform-specific code injected
2 `Setup.InitializeFirstChance` - a "first blood" placeholder for any steps you want to take before any of the later steps happen
3 `Setup.InitializeLastChance` - a "last ditch" placeholder for any steps you want to take after all of earlier steps have happened. Note that Android and iOS base classes

In this section, we'll only introduce some of the possibilities for creating and registering objects during initialize and setup. For a more in-depth introduction to MvvmCross IoC, see LINK-TODO.

###Registering cross-platform business objects in `App.Initialize`

It's very common for `App.Initialize` to include IoC registration. Indeed, the default nuget `App.cs` template includes

            CreatableTypes()
                .EndingWith("Service")
                .AsInterfaces()
                .RegisterAsLazySingleton();

This logic says:

- within the Assembly containing the `App`
- find all *creatable* `Type`s - meaning:
 - with `public` constuctors
 - not marked abstract
- find the interfaces they implement
- lazily register these types as singletons - meaning:
 - that no instance will be created until one is requested
 - that once the first instance has been created, then that same instance will be returned for all subsequent requests

Many other registration techniques are available - including registering objects from other assemblies, using instance-per-request registration and using individual rather than reflection-based registrations. For more on these, see LINK-TO-IOC-DOC-TODO.

###Registering platform-specific business objects in `Setup.InitializeFirstChance` and `Setup.InitializeLastChance`

These two placeholders provide key places for you to create and register services which are specific to your platform.

For example, if you wanted to implement an `EncryptionService` which would provide native data-encryption for your application, then you could do this during `Setup.InitializeFirstChance` using:

            Mvx.RegisterType<IEncryption, MyEncryption>();
       
This would then allow all of your `App` code - including code executed during `App.Initialize()` to use calls like:

            var encryption = Mvx.Resolve<IEncryption>();
            var safe = encryption.Encode(raw);

Alternatively, if you wanted to implement a `DialogService` which would be used during normal UI flow, then you might choose to register this during `Setup.InitializeLastChance` as:

            Mvx.RegisterSingleton<IDialogService>(new MyDialogService());

For many objects the choice of when to initialize - first or last - doesn't matter. For others, the key choice is whether the service needs to be available before or after the `App` is created and initialized.

##Changing trace/debug output

Each platform provides a virtual `CreateDebugTrace` methods which offers your application a chance to customise where `Mvx.Trace` messages are displayed.

To provide a custom trace implementation:

- first implement a class which provides `IMvxTrace`
- override `Setup.CreateDebugTrace()` in order to return an instead of your new class

One common use of this is simply to display messages to `Debug` using:

            public class MyDebugTrace : IMvxTrace
	{
		public void Trace(MvxTraceLevel level, string tag, Func<string> message)
		{
			Debug.WriteLine(tag + ":" + level + ":" + message());
		}

		public void Trace(MvxTraceLevel level, string tag, string message)
		{
			Debug.WriteLine(tag + ":" + level + ":" + message);
		}

		public void Trace(MvxTraceLevel level, string tag, string message, params object[] args)
		{
			try
			{
				Debug.WriteLine(string.Format(tag + ":" + level + ":" + message, args));
			}
			catch (FormatException)
			{
				Trace(MvxTraceLevel.Error, tag, "Exception during trace of {0} {1} {2}", level, message);
			}
		}
	}

this can be returned during `Setup` using:

            protected override IMvxTrace CreateDebugTrace() 
            { 
                        return new MyDebugTrace(); 
            }
            

##Changing the IoC container that MvvmCross uses

IoC is the first thing that MvvmCross setup starts.

This is done within the method `InitializeIoC` 

To override MvvmCross' IoC, you can:

- first find your alternative IoC implementation - e.g. something like AutoFac, Funq or TinyIoC
- then create an `Adapter` which maps the implementation behind an `IMvxIoCProvider` interface and which inhertis from `MvxSingleton<IMvxIoCProvider>` in order to provide a `Singleton`  
  - the majority of the adaption should be relatively straight-forwards - see `MvxSimpleIoCContainer` for how the default IoC container is provided.
  - The only *unusual* methods in the MvvmCross IoC interface are the `CallbackWhenRegistered` hooks - these provide callbacks when new object types are registered and may require a little custom code in the `RegisterXXX` methods within your adapter.
- finally, you can override `IMvxIoCProvider CreateIocProvider()` in your `Setup` class to return your IoC provider
 



#TODO

##Overriding ViewModel Location
##Custom IMvxAppStart
##Providing ValueConverters
##Providing CustomViews (android)
##FillBindingNames
##FillTargetFactories
##CustomPresenters

