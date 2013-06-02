---
layout: docs
categories: docs
title:  "Linker Include"
---

Data binding uses reflection and when using Monotouch (Xamarin.iOS) or Monodroid (Xamarin.Droid) their linker can sometimes be too aggresive in removing 'Unused' symbols. If you encounter NullReference exceptions when data binding then your going to need to add [fake references](https://raw.github.com/slodge/MvvmCross-Tutorials/master/Sample%20-%20TwitterSearch/TwitterSearch.UI.Touch/LinkerIncludePlease.cs):

{% highlight csharp %}
// LinkerIncludePlease.cs
[Preserve]
private class LinkerInclude
{
    private void IncludeStuff()
    {
        UITextField textField = null;
        textField.Text = textField.Text + "";

        UIButton button = null;
        button.TouchUpInside += delegate(object sender, EventArgs e) {			
        };
    }
}
{% endhighlight %}
