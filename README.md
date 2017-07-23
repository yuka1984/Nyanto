# Nyanto

Nyanto is a library that supports use of "DI" and use of "Rx" and "LifeCycle Management" at Xamarin Android. 

## concept

It is a framework created for making @yuka1984 easier when making Android applications with Xamarin Android.
Xamarin Android helps Autofac to use DI and ReactiveProperty.
We are also trying to realize correspondence with LifeCycle in relation to ReactiveProperty. This is affected by the implementation of ArchitectureComponents.

## Sample Application

github repository

https://github.com/yuka1984/DroidKaigi2017forXamarin/tree/master/src/DroidKaigi2017.Droid

Application movie

https://sleepyandhungry1984.tumblr.com/post/162514943270/xamarin-andorid-application

## How to use

### ApplicationBase

First, create an Application class that inherits from ApplicationBase.
ApplicationBase must abstract and implement the ContainerSetting method.

``` Protected abstract void ContainerSetting (ContainerBuilder builder) ``` 

With this method you set DI with Autofac.ContainerBuilder.

If you want to maintain the same lifecycle as the Application, use SingleInstance.

If you want to maintain the same lifecycle as the Activity, use InstancePerLifetimeScope will do.


### AppCompatActivityBase

When creating Activity, create it by inheriting AppCompatActivityBase.

AppCompatActivityBase gets the Autofac container from the Application class that inherits ApplicationBase, and performs BeginScope to create ILifetimeScope.

This created ILifetimeScope is maintained in the same lifecycle as the ViewModel of ArchitectureComponents. The concept of HolderFragment is used for implementation. By doing this, ILifetimeScope is maintained beyond Activity's Lifecycle and Dispose is done when the Activity is not fully used, and it ends.

The class inheriting AppCompatActivityBase automatically performs PropertyInjection. If classes registered in the container are necessary, if you implement them as Property, it will be injected. The timing when Injection is done is OnCreate. Before base.OnCreate is called, it is not injected.

Classes that inherit from AppCompatActivityBase must implement the ConfigurationAction method.

```Protected abstract void ConfigurationAction (ContainerBuilder containerBuilder);```

If you want to perform injection within the range of that Activity, register it here in Builder. It is assumed that you register mainly classes that require instances of Activity itself.

### FragmentBase

When creating Fragment, create it by inheriting FragmentBase.

Generic version for FragmentBase

Public abstract class FragmentBase <T>: FragmentBase where T: ViewModelBase

There are, and in most cases we will use that.

FragmentBase has T property ViewModel. Since it is a property, injection is done automatically if Autofac has registered ViewModel specified as Generic.

Although ViewModelBase will be described later, in most cases ViewModel will register with Autofac's InstancePerLifetimeScope so the same instance will be retained while the activity exists (not the instance cycle).

In short, it means that the ViewModel of the same instance will be injected no matter how many fragments are regenerated.

Classes that inherit from FragmentBase

``` Public abstract int ViewResourceId {get;}```

and

``` Protected abstract void Bind (View view);```

It is necessary to implement.

ViewResoureId should reply the resource ID of Fragment's ViewLayout.

In the Bind function, the View class of Fragment is passed as an argument to describe Bind of ViewModel and View.

Bind is called during Fragment's OnCreateView.

FragmentBase has a CompositDisposable field.

When using Rx, if you subscribe IDisposable in CompositDisposable, Dispose is done at the timing of Fragment.OnDetach and you can unsubscribe.

### ViewModelBase

We inherit ViewModelBase when creating ViewModel.

ViewModelBase is a common ViewModelBase implementing INotifyPropertyChanged, but there is one ingenuity to do LifeCycleManagement.

```protected ReadOnlyReactiveProperty <bool> IsActiveObservable;```

IsActiveObservable streams true when the Activity holding ViewModel changes to "START" or "RESUME". Otherwise it will false.

This allows you to handle lifecycle with ViewModel.
