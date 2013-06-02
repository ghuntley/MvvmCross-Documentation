---
layout: docs
categories: docs
title:  "Solution Layout"
---

We suggest the following logical naming pattern when laying out your solution:

![MvvmCross Solution Layout](images/solution-layout.png)

All application logic is stored within a core portable class library which is shared between and referenced in each specific platform application:

<div class="row">
  <div class="large-4 columns">
    <dl>
      <dt><h5>.Core</h5></dt>
      <dd>A portable class library that will be the heart of the application and where you will spend most of your time. <code>Profile104</code> is the recommended profile which should be used, you will need to select it when you create the project.</dd>
    </dl>
  </div>
  <div class="large-4 columns">
    <dl>
      <dt><h5>.Core.Tests</h5></dt>
      <dd>A NUnit class library that contains unit tests that confirm functionality of .Core.</dd>
    </dl>
  </div>
  <div class="large-4 columns">
    <dl>
      <dt><h5>.Droid</h5></dt>
      <dd>A monodroid (Xamarin.Android) application which contains Android user interface code for both phone and tablet. <strong>The <code>.Android</code> namespace prefix is reserved by Google for Android internals and must not be used.</strong></dd>
    </dl>
  </div>
</div>

<div class="row">
  <div class="large-4 columns">
    <dl>
      <dt><h5>.iOS</h5></dt>
      <dd>A monotouch (Xamarin.iOS) application which contains iOS user interface code for both iPhone and iPad. </dd>
    </dl>
  </div>
  <div class="large-4 columns">
    <dl>
      <dt><h5>.Mac</h5></dt>
      <dd>A monomac (Xamarin.Mac) application which contains OSX user interface code. </dd>
    </dl>
  </div>
  <div class="large-4 columns">
    <dl>
      <dt><h5>.WindowsPhone</h5></dt>
      <dd>A Windows Phone application which contains the user interface code.</dd>
    </dl>
  </div>
</div>

<div class="row">
  <div class="large-4 columns">
    <dl>
      <dt><h5>.WindowsStore</h5></dt>
      <dd>Windows 8 Store user interface code.</dd>
    </dl>
  </div>
  <div class="large-4 columns">
    <dl>
      <dt><h5>.Wpf</h5></dt>
      <dd>Windows Presentation Foundation user interface code.</dd>
    </dl>
  </div>
  <div class="large-4 columns">
    <!-- reserved for future expansion -->
  </div>
</div>