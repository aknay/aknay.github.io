---
layout: post
title:  "Calling API with Retrofit and Gson on Android"
date:   2018-1-08 10:00:00 +0800
excerpt_separator: <!-- excerpt -->
---
In this tutorial, we will create an Android application to call Github API using Retrofit and Gson.
<!-- excerpt -->

#### **Git Hub Project**
The complete project can be cloned from [here](https://github.com/aknay/Android-Tutorials/tree/master/RetrofitAndGson){:target="_blank"}.

#### **Prerequisites**
1. Please make sure Java and Android Studio IDE are installed on your system.
2. We will be using Kotlin Language instead of Java. Please make sure Kotlin plugin is installed also.

#### **Reference**
1. Excellent tutorials from [CodePath](https://guides.codepath.com/android/consuming-apis-with-retrofit){:target="_blank"}.


#### **Overview**
Retrofit converts REST API call into a Java interface. Once we get the callable objects from Retrofit, we will convert those objects to/from JSON using Gson (which is to serialize and deserialize Java objects from/to JSON).   


Briefly, our steps are
1. Look at Github API and JSON response
2. Convert JSON response into `class` that Gson can serialize/deserialize
3. Make Retrofit client with `GsonConverterFactory` 
4. Define Github API routes
5. Create service class to combine Retrofit client and Github API routes (which are defined in above two statements)
6. Finally, make API calls in `MainActivity` to test the implementation. 


#### **Add Dependencies**
In order to use Retrofit and Gson, we need these dependencies. 

```
    // Retrofit
    compile 'com.squareup.retrofit2:retrofit:2.3.0'

    // JSON Parsing
    compile 'com.google.code.gson:gson:2.8.2'
    compile 'com.squareup.retrofit2:converter-gson:2.3.0'
```

#### **Network Permisson**
We need to network permission to call API.
```
<uses-permission android:name="android.permission.INTERNET" />
```

#### **Convert GitHub User Json into Gson class**

Now, we will be converting JSON response from GitHub API into something that Gson can serialize/ deserialize. You can look at how to get a single user from [Github Doc](https://developer.github.com/v3/users/#get-a-single-user){:target="_blank"}.

At your browser, just type in the below statement. We are getting the username called `octocat` which is used in GitHub Docs.

```
https://api.github.com/users/octocat
```

You should get the following JSON raw response
```
{
  "login": "octocat",
  "id": 583231,
  "avatar_url": "https://avatars3.githubusercontent.com/u/583231?v=4",
  "gravatar_id": "",
  "url": "https://api.github.com/users/octocat",
  "html_url": "https://github.com/octocat",
  "followers_url": "https://api.github.com/users/octocat/followers",
  "following_url": "https://api.github.com/users/octocat/following{/other_user}",
  "gists_url": "https://api.github.com/users/octocat/gists{/gist_id}",
  "starred_url": "https://api.github.com/users/octocat/starred{/owner}{/repo}",
  "subscriptions_url": "https://api.github.com/users/octocat/subscriptions",
  "organizations_url": "https://api.github.com/users/octocat/orgs",
  "repos_url": "https://api.github.com/users/octocat/repos",
  "events_url": "https://api.github.com/users/octocat/events{/privacy}",
  "received_events_url": "https://api.github.com/users/octocat/received_events",
  "type": "User",
  "site_admin": false,
  "name": "The Octocat",
  "company": "GitHub",
  "blog": "http://www.github.com/blog",
  "location": "San Francisco",
  "email": null,
  "hireable": null,
  "bio": null,
  "public_repos": 7,
  "public_gists": 8,
  "followers": 2053,
  "following": 5,
  "created_at": "2011-01-25T18:44:36Z",
  "updated_at": "2018-01-01T12:31:09Z"
}

```

We will use [this website](http://www.jsonschema2pojo.org/){:target="_blank"} to convert JSON into class conveniently. So copy the JSON raw response above and paste it into the textbox. Select source type to `JSON` and Annotation style to `Gson`. Then click `Preview` and copy the class as Java. 

You should get the class with a long list of methods. Since we want to make 100% Kotlin, we will use `Ctrl+Alt+Shift+K` key at Android Studio to convert Java class to Kotlin class that will result in the following class.

{% highlight java linenos %}

package com.aknay.retrofit.model

import com.google.gson.annotations.Expose
import com.google.gson.annotations.SerializedName

class GitHubUser {

    @SerializedName("login")
    @Expose
    var login: String? = null
    @SerializedName("id")
    @Expose
    var id: Int? = null
    @SerializedName("avatar_url")
    @Expose
    var avatarUrl: String? = null
    @SerializedName("gravatar_id")
    @Expose
    var gravatarId: String? = null
    @SerializedName("url")
    @Expose
    var url: String? = null
    @SerializedName("html_url")
    @Expose
    var htmlUrl: String? = null
    @SerializedName("followers_url")
    @Expose
    var followersUrl: String? = null
    @SerializedName("following_url")
    @Expose
    var followingUrl: String? = null
    @SerializedName("gists_url")
    @Expose
    var gistsUrl: String? = null
    @SerializedName("starred_url")
    @Expose
    var starredUrl: String? = null
    @SerializedName("subscriptions_url")
    @Expose
    var subscriptionsUrl: String? = null
    @SerializedName("organizations_url")
    @Expose
    var organizationsUrl: String? = null
    @SerializedName("repos_url")
    @Expose
    var reposUrl: String? = null
    @SerializedName("events_url")
    @Expose
    var eventsUrl: String? = null
    @SerializedName("received_events_url")
    @Expose
    var receivedEventsUrl: String? = null
    @SerializedName("type")
    @Expose
    var type: String? = null
    @SerializedName("site_admin")
    @Expose
    var siteAdmin: Boolean? = null
    @SerializedName("name")
    @Expose
    var name: Any? = null
    @SerializedName("company")
    @Expose
    var company: Any? = null
    @SerializedName("blog")
    @Expose
    var blog: String? = null
    @SerializedName("location")
    @Expose
    var location: Any? = null
    @SerializedName("email")
    @Expose
    var email: Any? = null
    @SerializedName("hireable")
    @Expose
    var hireable: Any? = null
    @SerializedName("bio")
    @Expose
    var bio: Any? = null
    @SerializedName("public_repos")
    @Expose
    var publicRepos: Int? = null
    @SerializedName("public_gists")
    @Expose
    var publicGists: Int? = null
    @SerializedName("followers")
    @Expose
    var followers: Int? = null
    @SerializedName("following")
    @Expose
    var following: Int? = null
    @SerializedName("created_at")
    @Expose
    var createdAt: String? = null
    @SerializedName("updated_at")
    @Expose
    var updatedAt: String? = null

}

{% endhighlight %}

#### **Retrofit Client**
In order to call API and convert the response into Gson, we need a Retrofit client with Gson Converter. Therefore, we use `Retrofit Builder` with `GsonConverterFactory` for this class back. 

{% highlight java linenos %}
package com.aknay.retrofit.retrofit

import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory

object RetrofitClient {

    private var mRetrofit: Retrofit? = null
    private val BASE_URL = "https://api.github.com/"
    val client: Retrofit
        get() {
            if (mRetrofit == null) {
                return Retrofit.Builder()
                        .baseUrl(BASE_URL)
                        .addConverterFactory(GsonConverterFactory.create())
                        .build()
            }
            return mRetrofit!!
        }
}
{% endhighlight %}


#### **Define GitHub API Routes**

Here, we are defining routes for Github API. Since we only want to get a single user, we will be converting `GET /users/:username` API from Github Docs as this class. 

{% highlight java linenos %}
package com.aknay.retrofit.retrofit

import com.aknay.retrofit.model.GitHubUser

import retrofit2.Call
import retrofit2.http.GET
import retrofit2.http.Path

interface GitHubApiRoutes {
    @GET("/users/{username}")
    fun getUser(@Path("username") username: String): Call<GitHubUser>
}
{% endhighlight %}


#### **GitHub User Service**
After creating Retrofit Client and Routes, it is now time to combine these two and add into `Service` class.

{% highlight java linenos %}
package com.aknay.retrofit.retrofit

object GitHubUserService {
    val gitHubService: GitHubApiRoutes
        get() = RetrofitClient.client.create(GitHubApiRoutes::class.java)
}
{% endhighlight %}

#### **Main Activity**

Now what's left here is to call single user API call at MainActivity class.  

{% highlight java linenos %}
package com.aknay.retrofit

import android.os.Bundle
import android.support.v7.app.AppCompatActivity
import android.widget.Button
import android.widget.EditText
import android.widget.TextView
import com.aknay.mRetrofit.R
import com.aknay.retrofit.model.GitHubUser
import com.aknay.retrofit.retrofit.GitHubApiRoutes
import com.aknay.retrofit.retrofit.GitHubUserService
import retrofit2.Call
import retrofit2.Callback
import retrofit2.Response

class MainActivity : AppCompatActivity() {

    private var mGitHubApiRoutes: GitHubApiRoutes? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        mGitHubApiRoutes = GitHubUserService.gitHubService
        findViewById<Button>(R.id.button).setOnClickListener {
            val userName = findViewById<EditText>(R.id.editText)!!.text.toString()
            getUser(userName)
        }
    }

    private fun getUser(userName: String) {
        mGitHubApiRoutes!!.getUser(userName).enqueue(object : Callback<GitHubUser> {
            override fun onResponse(call: Call<GitHubUser>, response: Response<GitHubUser>) {
                if (response.isSuccessful) {
                    val msg = "Number of repo is : " + response.body()?.publicRepos
                    setTextView(msg)
                } else {
                    setTextView("User may not exist")
                }
            }

            override fun onFailure(call: Call<GitHubUser>, t: Throwable) {
                setTextView("Fail to call")
            }

        })
    }

    private fun setTextView(text: String) {
        findViewById<TextView>(R.id.textView)!!.text = text
    }
}
{% endhighlight %}


#### **Result**
1. Run with the emulator.
2. Type in the username to get the number of Repo.
3. If the user doesn't exist, you will get `User may not exist` response.