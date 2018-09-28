---
layout: post
title:  "How to open a CSV file from Android App"
date:   2018-09-28 10:00:00 +0800
excerpt_separator: <!-- excerpt -->
---
In this tutorial, we will look at how to open a CSV file from Android App and display the CSV data on App.  
<!-- excerpt -->

#### **Git Hub Project**
The complete project can be cloned from [here](https://github.com/aknay/Android-Tutorials/tree/master/ReadCsv){:target="_blank"}.


#### **Example CSV Files**

1. [First Example]({{site.url}}/assets/examples_csv/example.csv).
2. [Second Example]({{site.url}}/assets/examples_csv/example2.csv).

#### **Prerequisites**
1. Android Eumulator or Physical device
2. Android Studio
3. Kotlin Programming

#### **Reference**

1. [Official Android docs: Open files using storage access framework](https://developer.android.com/guide/topics/providers/document-provider){:target="_blank"}


#### **Overview**
In this tutorial, we will open a CSV file using Android storage access framework. Then read the content of the CSV file and load the data to display on the App. 

#### **Dependencies**
Here we don't need any extra dependencies. However, `minSdkVersion` must be at least `21` in order to use Android storage access framework. 

#### **Screenshot**

<figure>
  <div  class="image-container">
  
    <div class="image-list">
        <img src="{{ site.url }}/assets/device-2018-09-28-191317.png">
    </div>

    <div class="image-list">
        <img src="{{ site.url }}/assets/device-2018-09-28-191344.png">
    </div>

     <div class="image-list">
        <img src="{{ site.url }}/assets/device-2018-09-28-191407.png">
    </div>

  </div>

</figure>



#### **Main Activity**

At line 24, we use `setOnClickListener` to send `Intent` to open the Android built-in UI picker to open a file. Note that we are using `Intent.ACTION_OPEN_DOCUMENT` and `Intent.CATEGORY_OPENABLE` so that we can interactively browse through all the files that are `openable`. But we are limiting the openable files by using `intent.type = "text/*"` at line 27 so that we can only open text file.

Once the file is selected by a user, the function `onActivityResult` at line 32 will be invoked. Using the Uri from the incoming intent, we use `readCsv` function at line 40 to load a list of `String`. Once the string data is extracted from the CSV file, we use `joinToString` function to display on text view `mTextViewCsvResult`.

You can assign any number to `READ_REQUEST_CODE`. Doesn't has to be `123` in this example. But when you have multiple intents (eg. open image files, open PDF files) then you need to assign a unique number to each intent and act on it based on that number. 

{% highlight java linenos %}
package aknay.readcsv

import android.app.Activity
import android.content.Intent
import android.net.Uri
import android.os.Bundle
import android.support.v7.app.AppCompatActivity
import android.widget.Button
import android.widget.TextView
import java.io.BufferedReader
import java.io.IOException
import java.io.InputStreamReader


class MainActivity : AppCompatActivity() {

    private lateinit var mTextViewCsvResult: TextView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        mTextViewCsvResult = findViewById(R.id.textView_csvResult)

        findViewById<Button>(R.id.button_loadCsv)?.setOnClickListener {
            val intent = Intent(Intent.ACTION_OPEN_DOCUMENT)
            intent.addCategory(Intent.CATEGORY_OPENABLE)
            intent.type = "text/*"
            startActivityForResult(intent, READ_REQUEST_CODE)
        }
    }

    public override fun onActivityResult(requestCode: Int, resultCode: Int, resultData: Intent?) {
        if (requestCode == READ_REQUEST_CODE && resultCode == Activity.RESULT_OK) {
            resultData?.let { intent ->
                mTextViewCsvResult.text = readCSV(intent.data).joinToString(separator = "\n")
            }
        }
    }

    @Throws(IOException::class)
    fun readCSV(uri: Uri): List<String> {
        val csvFile = contentResolver.openInputStream(uri)
        val isr = InputStreamReader(csvFile)
        return BufferedReader(isr).readLines()
    }

    companion object {
        const val READ_REQUEST_CODE = 123
    }

}
{% endhighlight %}

#### **XML Layout**

We don't have much to explain here. Just take note of the `android:id` for `MainActivity`.

{% highlight java linenos %}
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/textView_csvResult"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="The result will be displayed here."
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <Button
        android:id="@+id/button_loadCsv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Load CSV File" />


</android.support.constraint.ConstraintLayout>
{% endhighlight %}