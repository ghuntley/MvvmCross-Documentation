

In each MvvmCross UI project there is a `Setup` class

This `Setup` is a bootstrapper for the MvvmCross system and for your app.

Internally this bootstrapper performa many steps. You can see most of them in the MvxSetup.cs class source which includes a sequence like this:

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

Added to these base class steps, each platform adds a small number of platform specific steps - eg Android adds some steps and methods for initialisation of the data-binding framework:

            InitializeSavedStateConverter()
            InitialiseBindingBuilder()
            MvxAndroidBindingBuilder CreateBindingBuilder()
            RegisterBindingBuilderCallbacks()
            FillBindingNames(IMvxBindingNameRegistry registry)
            FillAxmlViewTypeResolver(IMvxAxmlNameViewTypeResolver viewTypeResolver)
            FillNamespaceListViewTypeResolver(IMvxNamespaceListViewTypeResolver viewTypeResolver)
            FillValueConverters(IMvxValueConverterRegistry registry)
            FillTargetFactories(IMvxTargetBindingFactoryRegistry registry)

This long list of virtual methods does provide a lot of opportunities for overriding default MvvmCross behaviour. However, most applications actually override only very few of these methods. 

Some key ones you might want to be aware of are:

- `InitializeIoC` - use this is you want to change the start of IoC
- `InitializeFirstChance` - a "first blood" placeholder for any steps you want to take before any of the later steps happen
- `InitializeDebugServices` - a chance to initialize application trace - this can easily be customized - see http://stackoverflow.com/a/17234083/373321
- `InitializeLastChance` - a "last ditch" placeholder for any steps you want to take after all of earlier steps have happened. Note that Android and iOS base classes
