---
layout: docs
categories: docs
title:  "Application Start"
---

### Simple 
{% highlight csharp %}
public class App : MvxApplication
{
    public override void Initialize()
    {
        CreateableTypes()
          .EndingWith("Service")
          .AsInterfaces()
          .RegisterAsLazySingleton();

        RegisterAppStart.ViewModels.HomeViewModel>();
    }
}
{% endhighlight %}

### Conditional

{% highlight csharp %}
public class ConditionalStartup : MvxNavigationObject, IMvxAppStart
{
    public void Startup()
    {
        var applicationSettings = Mvx.Resolve<IAppSettings>();

        if (!applicationSettings.Authenticated)
            ShowViewModel<LoginViewModel>();
        else
            ShowViewModel<HomeViewModel>();
    }
}
{% endhighlight %}
