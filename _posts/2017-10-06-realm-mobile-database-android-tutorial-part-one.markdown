---
layout: post
title:  "Realm Database Tutorial for Android - Part One"
date:   2017-10-06 10:00:00 +0800
excerpt_separator: <!-- excerpt -->
---
In this tutorial, we will create a database using Realm Mobile Database and we will perform CRUD operations on the database.
<!-- excerpt -->

#### **Git Hub Project**
The complete project can be cloned from [here](https://github.com/aknay/Android-Tutorials){:target="_blank"}

#### **Prerequisites**
1. Please make sure Java and Android Studio IDE are installed on your system. 

#### **Reference**
1. [From Realm](https://realm.io/docs/java/latest/){:target="_blank"}

#### **Create An Android Application**
We will create a new Android application. So from Android Studio, choose `File -> New -> New Project`. Fill in with your prefer Application name and so on. Then choose `Empty Application` when you are asked for `Add an Activity to Mobile`. Finally, we just use the default name for `MainActivity` at `Configure Activity` Dialog. Then click `Finish`.  

#### **Add Dependencies**
You can check out thhe lastest release from [here](https://github.com/realm/realm-java/releases){:target="_blank"}.

Add this following line to `build.gradle(Module:app)` and click `Sync Now` at the top right corner.
```xml
  classpath "io.realm:realm-gradle-plugin:3.7.2"
```
Add this following line to `build.gradle(Module:app)` and click `Sync Now` at the top right corner.
```xml
  apply plugin: 'realm-android'
```

#### **Overall Package and Class layout**
This is the overall layout for our project. `Person` class is `Model` for our database and `PersonRepository` will do all the transition between our code and Realm Mobile Database. 
![Android Studio Dagger Dir]({{ site.url }}/assets/2017-10-06-android-studio-realm-mobile-database.png)


#### **Realm Database Application**
First, we need to initialize Realm Database once using `Realm.init(this);` . We can do it in this `RealmDatabaseApplication` Class.

{% highlight java linenos %}
import android.app.Application;
import io.realm.Realm;

public class RealmDatabaseApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        Realm.init(this);
    }
}
{% endhighlight %}
Since we create our customized application, we need to register this class to `AndroidManifest.xml`. Add this line between `application` tag.
```xml
android:name=".RealmDatabaseApplication"
```

Once you added the above line, the final `AndroidManifest.xml` will look like this.
{% highlight java linenos %}
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="aknay.realmdatabasetutorialpartone">

    <application
        android:name=".RealmDatabaseApplication"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>

{% endhighlight %}

#### **Crate Person Class**
1. create package called `model`
2. create class called `Person`
Please, look at the overall layout if you are not sure where to create them.

In order to Realm Database to handle the model, we need `Person` model class to extend `RealmObject`. Here we also use `@PrimaryKey` annotation is to indicate that the `id` is unique for a person instance. 

{% highlight java linenos %}
import io.realm.RealmObject;
import io.realm.annotations.PrimaryKey;

public class Person extends RealmObject {
    @PrimaryKey private long id;
    private String name;
    private int age;

    public void setId(long id) {
        this.id = id;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public long getId() {
        return id;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}

{% endhighlight %}

#### **Crate PersonRepository Class**
1. create package called `repository`
2. create class called `PersonRepository`

Again, please look at the overall layout if you are not sure where to create them.
This class handles all CRUD operations for the database. The class use `Singleton` pattern to get `PersonRepository` instance so that we will only instantiate only one time. Please note that calling `executeTransaction` function of `mRealm` also handle error itself. 

{% highlight java linenos %}
import android.support.annotation.NonNull;
import aknay.realmdatabasetutorialpartone.model.Person;
import io.realm.Realm;
import io.realm.RealmResults;

public class PersonRepository {
    private PersonRepository() {
        mRealm = Realm.getDefaultInstance();
    }

    private static PersonRepository mPersonRepository = null;
    private Realm mRealm = null;

    //singleton pattern
    public static PersonRepository getInstance() {
        if (mPersonRepository == null) {
            mPersonRepository = new PersonRepository();
        }
        return mPersonRepository;
    }

    public void deleteAll() {
        mRealm.executeTransaction(new Realm.Transaction() {
            @Override
            public void execute(@NonNull Realm realm) {
                realm.delete(Person.class);
            }
        });
    }

    public void insert(final Person person) {
        mRealm.executeTransaction(new Realm.Transaction() {
            @Override
            public void execute(@NonNull Realm realm) {
                realm.copyToRealm(person);
            }
        });
    }

    public RealmResults<Person> getAll() {
        return mRealm.where(Person.class).findAll();
    }

    public Person findByName(String name) {
        return mRealm.where(Person.class).equalTo("name", name).findFirst();
    }

    public void update(final Person person) {
        mRealm.executeTransaction(new Realm.Transaction() {
            @Override
            public void execute(@NonNull Realm realm) {
                Person personCopy = findByName(person.getName());
                personCopy.setAge(person.getAge());
            }
        });
    }
}
{% endhighlight %}

#### **MainActivity**
Now, this is the fun part. We first create two persons called 'Joe' and 'Bob'. Then we inserted to our database. Please take note that `id` has to be unique for each person as we set up the `id` as our `PrimaryKey` previously. After we added both of them, we list all people in our database. Then, we retrieve back the person by `Name`. Later we just change `Joe` age and observe the change.

{% highlight java linenos %}
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.util.Log;

import aknay.realmdatabasetutorialpartone.model.Person;
import aknay.realmdatabasetutorialpartone.repository.PersonRepository;
import io.realm.RealmResults;

public class MainActivity extends AppCompatActivity {
    private String TAG = this.getClass().getSimpleName();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        PersonRepository personRepository = PersonRepository.getInstance();
        
        //new person called 'Joe'
        Person joe = new Person();
        joe.setId(1); //unique ID
        joe.setAge(18);
        joe.setName("Joe");
        personRepository.insert(joe); //insert to repository

        //new person called 'Bob'
        Person bob = new Person();
        bob.setId(2);
        bob.setAge(21);
        bob.setName("Bob");
        personRepository.insert(bob);

        //list all people
        RealmResults<Person> results = personRepository.getAll();
        for (Person p : results) {
            Log.d(TAG, "Person List -> " + "name: " + p.getName() + ", " + "Age: " + p.getAge());
        }

        //retrieve 'Joe'
        Person retrievedJoe = personRepository.findByName("Joe");
        Log.d(TAG, "Young Joe -> " + "name: " + retrievedJoe.getName() + ", " + "Age: " + retrievedJoe.getAge());

        //update 'Joe' age
        joe.setAge(60);
        personRepository.update(joe);

        //observe the change
        Person oldJoe = personRepository.findByName("Joe");
        Log.d(TAG, "Old Joe -> " + "name: " + oldJoe.getName() + ", " + "Age: " + oldJoe.getAge());

        //delete all entries
        personRepository.deleteAll();
    }
}
{% endhighlight %}

#### **Result**
You should see this in log screen. 
```
Person List -> name: Joe, Age: 18
Person List -> name: Bob, Age: 21
Young Joe -> name: Joe, Age: 18
Old Joe -> name: Joe, Age: 60
```