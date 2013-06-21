---
layout: docs
categories: docs
title: MvvmCross objects
---

##MvvmCross high level objects

Deployed MvvmCross applications consist of two parts:

- the "core"  - containing all the ViewModels, Services, Models and 'business' code
- the "ui" - containing the Views and platform specific code for interacting with the "core"

For a multi-platform application, it's typical for there to be:

- a single "core" project, written as a PCL (Portable Class Library)
- a "ui" project per platform written as a native project for the current target platform.
- optionally some "plugins" - each one containing a PCL part and native parts - each one providing reusable abstractions of native functionality such as camera, geolocation, accelerometer, files, etc

This is the way that MvvmCross encourages people to write their applications, and this guide will . However, other approaches are possible - e.g. a single project can include both "core" and "ui", or multiple "core" projects can be written using copy-and-paste or using a technique such as file-linking

##Some key MvvmCross objects

There are a few key objects within an MvvmCross application:

- in the "core", there are:
 - an `App` - responsible for starting your ViewModels and your business logic
 - a `Start` object - responsible for deciding the first `ViewModel` or `ViewModels` which should be presented
 - one or more `ViewModels` - each one responsible for a piece of user interaction
 - your services, models, etc

- in each "ui", there are:
 - the native `Application` object - responsible for native lifecycle events - on each platform this object is a platform-specific class
 - an MvvmCross `Setup` class - resposnible for 'bootstraping' MvvmCross, your 'core' and your 'ui'
 - one or more `Views` - each one responsible for presenting one of your `ViewModels`
 - custom UI code - for controls, gestures, events, etc

##How an MvvmCross application starts

When an MvvmCross app starts on a native project, then:

- the native Application will 'be created' first
- within the construction of the native Application, a `Setup` will be created
 - the `Setup` will perform very basic tasks - e.g. initialisation of the IoC system (see LINK TO IOC)
 - then the `Setup` will call into the core project, construct an `App` and will call `Initialize` on it.
 - during the `Initialize` your App will typically: 
  - register your app-specific services with the IoC system
  - create and register a `Start` object
 - the `Setup` will then configure the UI side of the project - especially things like lookup tables for views
 - finally the `Setup` will start the MvvmCross binding engine (if needed)
- with `Setup` complete, your native Application can then actually start.
- to do this, it requests a reference to the `Start` object and call `Start()` on it 
- after this, the app will start presenting `ViewModels` using databound `Views`

##The MvvmCross Core

An MvvmCross Core project provides:

- an application object - typically in `App.cs`
- one or more ViewModels - normally in a folder called `ViewModels`
- your code: services, models, repositories, engines, units of work, etc - whatever your app needs to work
 
### App.cs

In each MvvmCross application there should be one and only one `App`

This `App` is not to be confused with the `ApplicationDelegate` in iOS, or with the `Application` objects in Android or Windows. Those native objects are there to provide the lifecycle of the native platform-specific code.

Instead, this `App` in MvvmCross is there to assist with the lifecycle of your ViewModels and your services, models, etc

The key method within an `App` are:

- `Initialize` - called on start up
- `FindViewModelLocator(MvxViewModelRequest request)` - used to find the object which provides `ViewModels` during navigation

The specific jobs that your `App` should do during its `Initialize` are:

- to construct and/or IoC-register any objects specific to your applications - services, models, etc
- to register an `IMvxAppStart` object

A default `App` supplied via nuget, looks like:

    using Cirrious.CrossCore.IoC;

    namespace MyName.Core
    {
        public class App : Cirrious.MvvmCross.ViewModels.MvxApplication
        {
            public override void Initialize()
            {
                CreatableTypes()
                    .EndingWith("Service")
                    .AsInterfaces()
                    .RegisterAsLazySingleton();
  			
                RegisterAppStart<ViewModels.FirstViewModel>();
            }
        }
    }

This `App`:

- within Initialize
 - looks within the current `Assembly` (the "core" Assembly) and uses Reflection to register all classes ending in `Service` as lazily-constructed singletons
 - call `RegisterAppStart<TViewModel>` to create and register a very simple `IMvxAppStart` implementation - an implementation which always shows a single `FirstViewModel` when `Start()` is called
- uses the default `ViewModelLocator` - this default uses naming conventions to locate and construct `ViewModels` and creates a new `ViewModel` for each and every request from a `View`

If you wanted to use a custom ImvxAppStart object, see TODO.

### ViewModels

In each MvvmCross 'core' application your `ViewModels` provide containers for the state and the behaviour for your User Inteface.

Typically they do this using:

- C# Properties
- the `INotifyPropertyChanged` and `INotifyCollectionChanged` interfaces to send notifications when properties change
- special `ICommand` properties which can allow View events (e.g. button taps) to call actions within the `ViewModel`
    
For MvvmCross, `ViewModels` normally inherit from `MvxViewModel`

A typical `ViewModel` might look like:

    public class FirstViewModel 
        : MvxViewModel
    {
        private string _name;
        public string Name
        {
            get { return _name; }
            set { _name = value; RaisePropertyChanged(() => Name); }
        }
        
        private readonly MvxCommand _resetCommand;
        public ICommand ResetCommand
        {
            get 
            {
                _resetCommand = _resetCommand ?? new MvxCommand(() => Reset());
                return _resetCommand;
            }
        }
        
        private void Reset()
        {
            Name = string.Empty;
        }
    }

This `FirstViewModel` has:

- a single `Name` property which raises a `PropertyChanged` notification when it changes
- a single `ResetCommand` command which will call the `Reset()` method whenever the command is executed.
 
Beyond this simple example, `ViewModels` can also:

- contain dynamic lists (TODO link to ObservableCollections)
- be constructed from IoC (TODO link to CIRS)
- use 'techniques' like `MvxCommandCollection`, `IMvxINPCInterceptor` and Fody to remove some of the boilerplate code (TODO - links)

##The MvvmCross UI

An MvvmCross 'ui' project provides:

- the native platform-specific application code - e.g `Main.cs` and `AppDelegate.cs` on Xamarin.iOS
- a `Setup.cs` class
- one or more `Views` - each one responsible for presenting one of your `ViewModels`
- custom UI code - for controls, gestures, events, etc

###Platform specific application code

####iOS

On iOS, we need to replace the normal `AppDelegate.cs` class with an `MvxApplicationDelegate`

An initial replacement looks like:

  using MonoTouch.Foundation;
	using MonoTouch.UIKit;
	using Cirrious.CrossCore;
	using Cirrious.MvvmCross.Touch.Platform;
	using Cirrious.MvvmCross.ViewModels;

	namespace MyName.Touch
	{
		[Register ("AppDelegate")]
		public partial class AppDelegate : MvxApplicationDelegate
		{
			UIWindow _window;

			public override bool FinishedLaunching (UIApplication app, NSDictionary options)
			{
				_window = new UIWindow (UIScreen.MainScreen.Bounds);

				var setup = new Setup(this, _window);
				setup.Initialize();

				var startup = Mvx.Resolve<IMvxAppStart>();
				startup.Start();

				_window.MakeKeyAndVisible ();
			
				return true;
			}
		}
	}
  
####Android

On Android, we don't normally have any `Application` to override. Instead of this, MvvmCross by default provides a `SplashScreen` - this typically looks like:

    using Android.App;
    using Android.Content.PM;
    using Cirrious.MvvmCross.Droid.Views;

    namespace CustomBinding.Droid
    {
        [Activity(
  	    Label = "CustomBinding.Droid"
		    , MainLauncher = true
		    , Icon = "@drawable/icon"
		    , Theme = "@style/Theme.Splash"
		    , NoHistory = true
		    , ScreenOrientation = ScreenOrientation.Portrait)]
        public class SplashScreen : MvxSplashScreenActivity
        {
            public SplashScreen()
                : base(Resource.Layout.SplashScreen)
            {
            }
        }
    }
    
Importantly, please note that this class is marked with `MainLauncher = true` to ensure that this is the first thing created when the native platform starts.

####WindowsPhone

On WindowsPhone, a new project will contain a native `App.xaml.cs`

To adapt this for MvvmCross, we simply:

1. add a line to the constructor:

            var setup = new Setup(RootFrame);
            setup.Initialize();

2. add a block to `Application_Launching` to force the native app to defer the start actions to `IMvxAppStart`
 
        private void Application_Launching(object sender, LaunchingEventArgs e)
        {
            RootFrame.Navigating += RootFrameOnNavigating;
        }

        private void RootFrameOnNavigating(object sender, NavigatingCancelEventArgs args)
        {
            args.Cancel = true;
            RootFrame.Navigating -= RootFrameOnNavigating;
            RootFrame.Dispatcher.BeginInvoke(() => { Cirrious.CrossCore.Mvx.Resolve<Cirrious.MvvmCross.ViewModels.IMvxAppStart>().Start(); });
        }

####WindowsStore

On WindowsStore, a new project will again contain a native `App.xaml.cs`

To adapt this for MvvmCross, we simply find the method `OnLaunched` and replace the `if (rootFrame.Content == null)` block with:

                var setup = new Setup(rootFrame);
                setup.Initialize();
                
                var start = Cirrious.CrossCore.Mvx.Resolve<Cirrious.MvvmCross.ViewModels.IMvxAppStart>();
                start.Start();

###Setup.cs

The Setup class is the bootstrapper for the MvvmCross system.

This bootstrapper goes through a lot of steps. You can see most of them in the MvxSetup.cs class source which includes a sequence like this:

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

STAR HERE
Many applications override very few methods. However, some key steps you might want to be aware of are:

- `InitializeIoC` - use this is you want to change the start of IoC
- `InitializeFirstChance` - a "first blood" placeholder for any steps you want to take before any of the later steps happen
- `InitializeDebugServices` - a chance to initialize application trace - this can easily be customized - see http://stackoverflow.com/a/17234083/373321
