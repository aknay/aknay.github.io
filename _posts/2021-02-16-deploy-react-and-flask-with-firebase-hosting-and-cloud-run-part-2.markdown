---
layout: post
title:  "Deploy React and Flask With Firebase Hosting and Cloud Run - Part 2"
date:   2021-02-16 21:00:00 +0800
excerpt_separator: <!-- excerpt -->
---

This is the tutorial guide to host a React + Flask App with Firebase Hosting and Cloud Run. 

<!-- excerpt -->

## **Series**
There are two parts to this series. Please read [here]({% post_url 2021-02-16-deploy-react-and-flask-with-firebase-hosting-and-cloud-run-part-1%}) for Part 1.

# Overview
In this tutorial, we will first build and deploy our back-end app on Cloud Run. Then we will add the back-end API on our `React` app. Finally, we will deploy our front-end app to Firebase Hosting and test the functionality. 

# Prerequisites
1. Linux OS
2. Familiar with JavaScript
3. Familiar with Python
4. Familiar with React
5. Node.js and Python 3 are installed
6. Docker Engine is installed

Please remember that this is the folder structure we created in Part 1.
<pre><font color="#3465A4"><b>.</b></font>
├── <font color="#3465A4"><b>back-end</b></font>
└── <font color="#3465A4"><b>front-end</b></font>
    └── <font color="#3465A4"><b>fire-base-hosting-react-app</b></font></pre>

<br>

# Back-end (Cloud Run)

## Before you begin Cloud Run
We need to do the following items before we could deploy our code on Cloud Run.
1. Set up billing for your project
2. Install the Cloud SDK.

#### 1. Set up billing for your project
For this first item, basically, we need a valid payment method to use Google Cloud. But we can use Cloud Run within the free tier. Refer [this link](https://cloud.google.com/billing/docs/how-to/modify-project) to set up the project.  Once you created a project at the cloud console, we will get a unique `Project-ID` and we will use it in the next step.

#### 2. Install and setup gcloud
Please access to [this link](https://cloud.google.com/sdk/docs/install#deb) for `gcloud` installation.
> If you are using `snap`, go to this link for `https://cloud.google.com/sdk/docs/downloads-snap` for quick installation.


## Build and Deploy on Cloud Run
Let's continue from the folder we created in Part 1. So, 
```
cd back-end
```
Here, we need the `PROJECT-ID` you created on Google Cloud Console. You can get it by running `gcloud config get-value project`.
Now we can build our container image using Cloud Build.

```
gcloud builds submit --tag gcr.io/<YOUR-PROJECT-ID>/backend
```
Once finished building, we can deploy our container image to Cloud Run.
```
gcloud run deploy --image gcr.io/<YOUR-PROJECT-ID>/backend --platform managed
```
After successful deployment, you will be given a link on your console. It will be something like `https://<UNIQUE-LINK-OF-YOUR-PROJECT>run.app`. Once you access the link with `YOUR-UNIQUE-LINK/time`, you will see the `time` value in JSON. Now our back-end is working well.

<br>

# Front-End (Firebase Hosting)

## Before you begin Firebase Hosting
1. Create a Project at Firebase Console
2. Install and set up Firebase CLI 

#### 1. Create a Project at Firebase Console
We only need to create a new project at [Firebase Console](https://console.firebase.google.com/). Just remember the project ID that you just created as we will need this soon.

#### 2. Install and set up Firebase CLI
Please refer to [this link](https://firebase.google.com/docs/cli) for the detailed installation.
You can just install Firebase CLI by
```
npm install firebase-tools -g
```

## Build and Deploy on Firebase Hosting

This time, we will host our `React` app in Firebase Hosting.
 
### Step 1. Build React App
 Once we set up Firebase CLI, we need to build our `react-app`. Let's go back to the react-app folder. Make sure we are at `<OUR-PROJECT>/front-end/fire-base-hosting-react-app`. 
 
  Now let's point our `time` API to the back-end at the cloud instead of the local server. Now change the link in `App.js` from `http://localhost:8080/time` to `https://<UNIQUE-LINK-OF-YOUR-BACK-END-PROJECT>run.app/time` during the `fetch` statement.
  
 ```js
 import React, { useState, useEffect } from 'react';
import './App.css';

function App() {

  const [currentTime, setCurrentTime] = useState(0);

  useEffect(() => {
    fetch('https://<UNIQUE-LINK-OF-YOUR-BACK-END-PROJECT>run.app/time').then(res => res.json()).then(data => {
      setCurrentTime(data.time);
    });
  }, []);

  return (
    <div className="App">
      <header className="App-header">
        <p>The current time is {currentTime}.</p>
      </header>
    </div>
  );
}

export default App;

 ```
 Now we build our react-app.  Once we build that, static files will be generated in the `build` folder.
 ```
 npm run build
 ```

### Step 2. Deploy to Firebase Hosting
Let's deploy to Firebase Hosting.
We first init our Firebase Hosting at CLI.
```
firebase init hosting
```
Then choose the existing project that you just created
```
> Use an existing project
```
When asked to use the public directory, type in `build` as we will use the `build` folder we just created from building the react-app.
```
? What do you want to use as your public directory? (public)
```
just `y` to this following question.
```
? Configure as a single-page app (rewrite all urls to /index.html)? (y/N)
```
just `n` for this following question as we don't want to set up with GitHub now.
```
? Set up automatic builds and deploys with GitHub? (y/N)
```
Then type `n` for the following question as we don't want to override the `build/index.html` which is generated from react-app.
```
File build/index.html already exists. Overwrite? (y/N)
```

Finally, deploy for front-end by
```
firebase deploy
```

You will be given a new link for the front-end like this `https://<YOUR-FRONT-END-PROJECT-ID>.web.app`. Access that link and you see see `time` value changed every time you refresh the website. Now your front-end react-app will call `time` API from the back-end whenever you refresh the page. Fantastic.

# Reference
1. [Cloud Run Quick Start Guide](https://cloud.google.com/run/docs/quickstarts/build-and-deploy)
2. [Hosting Flask servers on Firebase from scratch](https://medium.com/firebase-developers/hosting-flask-servers-on-firebase-from-scratch-c97cfb204579)
