---
layout: post
title:  "Access Firebase Cloud Firestore using Google Cloud Functions - Part 2"
date:   2020-02-03 10:05:00 +0800
excerpt_separator: <!-- excerpt -->
---
In this tutorial, we will write cloud functions and make API requests. Later we will deploy the cloud functions to Firebase.    
<!-- excerpt -->

## **Series**
There are two parts to this series. Please read [here]({% post_url 2020-02-03-access-firebase-cloud-firestore-using-google-cloud-functions-part-1%}) for Part 1.

## **Prerequisites**

1. You need to know how to create a Firebase project; You can refer to the starter guide [here](https://firebase.google.com/docs/firestore/quickstart).
2. Familiar with JavaScript/TypeScript
3. Familiar with Python

## **Overview**
In this tutorial, we will first write `cloud functions` to access `Cloud Firestore`. Then we will write `python` to make API requests to cloud functions that we had written. After that, we will verify the output result of each function call. Finally, we will deploy those cloud functions to Firebase.

## **Writing Cloud Functions**

From the project that you just created, open `index.ts` file under `YOUR_PROJECT/functions/src`. Let's start writing code.

We have two functions here.
1. `export const addUser`
2. `export const getUser`


We will be using `json` when we are adding the user name. Therefore, you have to set the `content-type` to `application/json`. Then we will be adding in `users` collection as shown in this line `const docRef = db.collection("users").doc(name);`. We also add server time by using this line `FieldValue.serverTimestamp()` so that we know when the user is added to the system. 

When we need to user back, we just get it back from `users` collection also. You can also see that there are `console.log` to help you to debug if you have any problem getting it right.

``` python
import * as functions from "firebase-functions";
import * as admin from "firebase-admin";

admin.initializeApp();

export const addUser = functions.https.onRequest((request, response) => {
  console.log(request.get("content-type"));
  console.log('the request name' + request.body.name);

  const name = request.body.name;
  switch (request.get("content-type")) {
    case "application/json":
    const db = admin.firestore();
      const FieldValue = require("firebase-admin").firestore.FieldValue;
      const docRef = db.collection("users").doc(name);
      docRef
        .set({
          name: name,
          added_on: FieldValue.serverTimestamp()
        })
        .catch(error => {
          console.log(error);
          response.status(500).send(error);
        });
      break;
  }
  response.status(200).send(`${name} has been added`);
});

export const getUser = functions.https.onRequest((request, response) => {
  console.log('the request name' + request.body.name);
  const db = admin.firestore();
  const name = request.body.name;
  db.collection("users")
    .doc(name)
    .get()
    .then(snapshot => {
      response.send(snapshot.data());
    })
    .catch(error => {
      console.log(error);
      response.status(500).send(error);
    });
});
```

## **Complie Cloud Functions**

Before, we call any cloud functions, we need to compile the code from the previous section. Make sure you are in 
`<YOUR_PROJECT>/functions` folder. 

<ol>
<li>
    To check any error/subtle bug in your code<code class="highlighter-rouge">index.ts</code> file. Make sure there is no error after you run it.
  <pre class="prettyprint"><code class="language-sh">$ npm run-script lint</code></pre>
</li>
<li>
    To compile your code and generate a native JavaScript file. 
  <pre class="prettyprint"><code class="language-sh">$ npm run-script build</code></pre>
</li>

<li>
    Now we are going to run those functions that we wrote. Take note of two links given at the console after you run it. Those functions will be running locally (You can see that those links started with <code class="highlighter-rouge">localhost</code>) but <code class="highlighter-rouge">firebase admin</code> will be accessing your Cloud Firestore database from Google.
  <pre class="prettyprint"><code class="language-sh">$ firebase serve --only functions</code></pre>
</li>
</ol>

## **Call Cloud Functions**

Calling cloud function can be done by using `curl` but I think it is a little bit harder to see with `json` data. So we will be using `python` and `requests` to call those functions. Please make sure you have installed `requests` via `pip`.

#### 1. To Add User
Make sure you change `url` with the generated link from the console you run it before. We are going to add `Jane` as an user to our database.
``` python
import requests
url = 'http://localhost:5000/<YOUR PROJECT LINK>/addUser'
payload = '{"name":"Jane"}'
headers = {'content-type': 'application/json', 'Accept-Charset': 'UTF-8'}
r = requests.post(url, data=payload, headers=headers)
print(r.content)
```
#### Result
You should see the following output when the user is added successfully. 
``` python

b'Jane has been added'

```


#### 2. To Get User
Again make sure you change `url` with the generated link from the console you run it before. This time, we should get the user back.
``` python
import requests
url = 'http://localhost:5000/<YOUR PROJECT LINK>/getUser'
payload = '{"name":"Jane"}'
headers = {'content-type': 'application/json', 'Accept-Charset': 'UTF-8'}
r = requests.post(url, data=payload, headers=headers)
print(r.content)
```
#### Result
You should see the following output when the request is successful. Of course, the timestamp will be different. 
``` python

b'{"name":"Jane","added_on":{"_seconds":1580635800,"_nanoseconds":625000000}}'

```

## **Deploy Cloud Functions**
You can deploy your cloud functions to Google server at your `<YOUR_PROJECT>/functions` by typing the following command at your console. You should see that links are generated. You can use those links to access remotely. Just replace your `url` in `Python` code from above and verify your output.
  <pre class="prettyprint"><code class="language-sh">$ firebase deploy</code></pre>


> NOTE: Take care of your Firebase security in general. Anything access from the outside world can incur charges if you exceed certain thresholds. If you don't need it, just delete the functions; just to be safe. You can delete functions at Firebase console.   

## **Conclusion**
We first set up `gcloud SDK` and `Firebase CLI` at [Part 1]({% post_url 2020-02-03-access-firebase-cloud-firestore-using-google-cloud-functions-part-1%}). Then we wrote `cloud functions` and access `Cloud Firestore` with `Python` and verify our results. After this part is done, we deploy cloud functions to Firebase to be able to access globally. With this, you should able to write and access cloud functions using Firebase.    

