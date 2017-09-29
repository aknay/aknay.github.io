---
layout: post
title:  "DI Injection with Dagger 2 - Hello World Example for Android"
date:   2017-09-29 10:00:00 +0800
excerpt_separator: <!-- excerpt -->
---
In this tutorial, we will create a simplest Dagger 2 - Hello World Example for Android using Android Studio.  
<!-- excerpt -->

#### **Git Hub Project**
The complete project can be cloned from [here](https://github.com/aknay/dagger2-hello-world-android-example){:target="_blank"}

#### **Prerequisites**
1. Please make sure Java and Android Studio IDE are installed on your system. 

#### **Reference**
1. [From Code Path](https://github.com/codepath/android_guides/wiki/Dependency-Injection-with-Dagger-2){:target="_blank"}
2. [Slides](https://docs.google.com/presentation/d/1bkctcKjbLlpiI0Nj9v0QpCcNIiZBhVsJsJp1dgU5n98/edit#slide=id.gf37ec81d4_1_0){:target="_blank"}

#### **Create An Android Application**
We will create a new Android application. So from Android Studio, choose `File -> New -> New Project`. Fill in with your prefer Application name and so on. Then choose `Empty Application` when you are asked for `Add an Activity to Mobile`. Finally, we just use the default name for `MainActivity` at `Configure Activity` Dialog. Then click `Finish`.  

#### **Add Dependencies**
Add these following lines to `build.gradle(Module:app)` and click `Sync Now` at the top right corner.
```xml
    compile 'com.google.dagger:dagger:2.11'
    annotationProcessor 'com.google.dagger:dagger-compiler:2.11'
```

#### **Create Packages, Classes and Interface**
Now, we will just create empty classes and interface, just want to explain how these are arranged in Dagger 2. 
1. First, create a package called `Dagger`
2. Under `Dagger` package, add `AppModule` class
3. Under `Dagger` package, add `AppComponent` interface
4. Under `Dagger` package, add `DaggerTestApp` class

1. First, create a package called `Pojo`
2. Under `Pojo` package, add `DaggerTestClass` class


Finally, the directory should look like this

![Android Studio Dagger Dir]({{ site.url }}/assets/2017-09-29-android-studio-dagger-dir.png)

#### **Understanding How They Link Together**


##### Real World Example
<div class="mermaid">
graph LR;
   A(Network Client)
   B(Cloud Api)
   C(Network Module)

   D(Database Client)
   E(Storage Module)

   F(Application Component)
   G(Component Builder)
   H(Activities<br /> Fragments<br /> Objects)
    A --> C
    B --> C
    D --> E
    C --> F
    E --> F
    F --> G
    G --> H

    style A fill:#27ae60  ,stroke:#333;
    style B fill:#27ae60  ,stroke:#333;
    style D fill:#27ae60  ,stroke:#333;
    style C fill:#8ed861  ,stroke:#333;
    style E fill:#8ed861  ,stroke:#333;
    style F fill:#7070ff  ,stroke:#333;
    style G fill:#2e4f70  ,stroke:#333;
    style H fill:#ee5133  ,stroke:#333;
</div>

##### In Our Example
<div class="mermaid">
graph LR;
   A(DaggerTestClass)-->B(AppModule)
      B --> C(AppComponent)
      C --> D(DaggerTestApp)
      D --> E(MainActivity)
   style A fill:#27ae60  ,stroke:#333;
   style B fill:#8ed861  ,stroke:#333;
   style C fill:#7070ff  ,stroke:#333;
   style D fill:#2e4f70  ,stroke:#333;
   style E fill:#ee5133  ,stroke:#333;
</div>

From the above diagram, the `DaggerTestClass` is the one that we want to inject. Therefore, the `DaggerTestClass` will be instantiated in `AppModule`. Then `AppModule` will be added to `AppComponent` to define which module to use and where to `inject` them. Remember, there can be multiple modules to load and multiple places to inject (MainActivity, Fragments...), just like in real world example. Once we lay out how the injection will be done, we use `DaggerTestApp` to build and generate code for us. After code generation is done, we can inject them in our `MainActivity`. That's our plan. 

#### **Start Coding Our Plan**

##### DaggerTestClass.java
This is just plain POJO class. This will be injected to our `MainActivity`.
{% highlight java linenos %}
package aknay.daggertestapp.Pojo;

public class DaggerTestClass {
    public String getString(String s) {
        return "Hello " + s;
    }
}
{% endhighlight %}

##### AppModule.java
You can identify the `Module` class by annotation `@Module`. In this class, we use `@Provides` to provide `DaggerTestClass` instance. The `@Singleton` is used here so that there will only one instance of `DaggerTestClass`. 
{% highlight java linenos %}
package aknay.daggertestapp.Dagger;

import android.app.Application;
import javax.inject.Singleton;
import aknay.daggertestapp.Pojo.DaggerTestClass;
import dagger.Module;
import dagger.Provides;

@Module
public class AppModule {

    Application mApplication;

    public AppModule(Application application) {
        mApplication = application;
    }

    @Provides
    @Singleton
    DaggerTestClass provideDaggerTestClass() {
        return new DaggerTestClass();
    }
}
{% endhighlight %}

##### AppComponent.java
You might notice that we use `@Component` annotation to load `AppModule.class`. You can add multiple modules here. In this interface, we use `inject` for the injection at `MainActivity`. You can also inject it to your `Fragment`s. 
{% highlight java linenos %}
package aknay.daggertestapp.Dagger;

import javax.inject.Singleton;
import aknay.daggertestapp.MainActivity;
import dagger.Component;

@Singleton
@Component(modules = {AppModule.class})
public interface AppComponent {
    void inject(MainActivity activity);
}
{% endhighlight %}

##### DaggerTestApp.java
This is the final part before we start injection. This is also the part for Dagger code generation of our setup. In line `13`, we use `DaggerAppComponent` to build. This is the only naming you need to follow. Because we name our application component as `AppComponent`, we need to use `Dagger` + `AppComponent` to build. Once you write that into Android Studio, the IDE will show you as an error as the `DaggerAppComponent` is not recognized. That's because Dagger hasn't generated the code for us yet. Therefore, we generate the code by `Build -> Make Project` from the main menu. After that, everything will be green except `appModule` is deprecated. It's just a warning and it will be gone after we start using that component in our `MainActivity`.  
{% highlight java linenos %}
package aknay.daggertestapp.Dagger;

import android.app.Application;

public class DaggerTestApp extends Application {
    private AppComponent mAppComponent;

    @Override
    public void onCreate() {
        super.onCreate();

        //Must Follow Dagger<Your App Component Name>
        mAppComponent = DaggerAppComponent.builder()
                .appModule(new AppModule(this))
                .build();
    }

    public AppComponent getAppComponent() {
        return mAppComponent;
    }
}
{% endhighlight %}

##### AndroidManifest.xml
Before we do the injection, we need to declare `DaggerTestApp` as `Application`. Otherwise, it won't be instantiated.
Add the following line to `AndroidManifest.xml` after the `<application` tag.
```
android:name=".Dagger.DaggerTestApp"
```

##### MainActivity.java
In this `MainActivity`, we will finally inject our `DaggerTestClass` using `@Inject` annotation. After that, we use `DaggerTestApp` application and perform the injection in our `MainActivity`. Then we just use the `mDaggerTestClass` instance to print out using Logger.

{% highlight java linenos %}
package aknay.daggertestapp;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import javax.inject.Inject;
import aknay.daggertestapp.Dagger.DaggerTestApp;
import aknay.daggertestapp.Pojo.DaggerTestClass;

public class MainActivity extends AppCompatActivity {
    @Inject DaggerTestClass mDaggerTestClass;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ((DaggerTestApp) getApplication()).getAppComponent().inject(this);
        setContentView(R.layout.activity_main);
        Log.d("MainActivity", mDaggerTestClass.getString("Dagger 2"));
    }
}
{% endhighlight %}

#### **Result**
You should see this in log screen. 
```
Hello Dagger 2
```