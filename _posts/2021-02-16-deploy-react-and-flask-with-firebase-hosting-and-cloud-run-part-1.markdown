---
layout: post
title:  "Deploy React and Flask With Firebase Hosting and Cloud Run - Part 1"
date:   2021-02-16 20:00:00 +0800
excerpt_separator: <!-- excerpt -->
---

This is the tutorial guide to create a React + Flask App and how to test docker locally. 

<!-- excerpt -->

## **Series**
There are two parts to this series. Please read [here]({% post_url 2021-02-16-deploy-react-and-flask-with-firebase-hosting-and-cloud-run-part-2%}) for Part 2.

# Overview
In this tutorial, we will first create a `Flask` project and containerize it as an app. Then we run the app to test the back-end functionality. After that, we will create a `React` app with the API from the back-end. Once all of them are done, we will test again from the back-end to the front-end locally.

# Prerequisites
1. Linux OS
2. Familiar with JavaScript
3. Familiar with Python
4. Familiar with React
5. Node.js and Python 3 are installed
6. Docker Engine is installed


# Setup Folders
We want to create both front-end and back-end folder inside a folder
```
$ mkdir deploy_react_and_flask_with_firebase_hosting_and_cloud_run
$ cd deploy_react_and_flask_with_firebase_hosting_and_cloud_run
$ mkdir front-end
$ mkdir back-end
```
The final folder structure will be like the following.
<pre><font color="#3465A4"><b>.</b></font>
├── <font color="#3465A4"><b>back-end</b></font>
└── <font color="#3465A4"><b>front-end</b></font>
    └── <font color="#3465A4"><b>fire-base-hosting-react-app</b></font></pre>

> We don't need to create a `fire-base-hosting-react-app` folder ourselves. This folder will be created with the `npx create-react-app` command later on.

# Back-end (Local)
Let's create a back-end first
We go into the back-end folder we just created
```
cd back-end
```
### Step 1: Write the sample application
Then create a `main.py` and paste the code from the below. Here we need to use`CORS`. Otherwise, The front-end cannot make the call to the back-end.
``` py
import os
import time

from flask import Flask
from flask_cors import CORS

app = Flask(__name__)
cors = CORS(app)


@app.route('/time')
def get_current_time():
    return {'time': time.strftime("%I:%M:%S %p", time.localtime())}


if __name__ == "__main__":
    app.run(debug=True, host="0.0.0.0", port=int(os.environ.get("PORT", 8080)))
```
### Step 2: Write package list
Create a `requirements.txt` and paste the code from the below. These packages are needed for the back-end.
```
flask
gunicorn
flask-cors
```
### Step 3: Containerize an app
To containerize this app, create a new file named `Dockerfile` and copy the following content.
```
FROM python:3.8

## Copy local code to the container image.
ENV APP_HOME /app
WORKDIR $APP_HOME
COPY . ./

# Install Packages
RUN pip install -r requirements.txt

# Set an environment variable for the port for 8080
ENV PORT 8080

# Run gunicorn bound to the 8080 port.
CMD exec gunicorn --bind :$PORT --workers 1 --threads 8 main:app
```
Let's build a local docker and test first.
```
sudo docker build --tag backend:python .
```
Then run that docker.
```
sudo docker run --rm -p 8080:8080 -e PORT=8080 backend:python
```

When you access `http://localhost:8080/time` on your browser, the `time` value will be shown.

# Front-end (Local)
We leave from the back-end and go into the `front-end` folder.
```
cd ..
cd front-end
```

### Step 1: Create React-App application

Now we create a react-app using the following command.
```
npx create-react-app fire-base-hosting-react-app
cd fire-base-hosting-react-app
```

Let's change the `App.js` file inside the `src` folder with the following code.

> Take note here is that we use `fetch('http://localhost:8080/time')` in `App.js` to get `time` value from the local server for now. It will be changed once we deploy to the cloud.

```js
import React, { useState, useEffect } from 'react';
import './App.css';

function App() {
  const [currentTime, setCurrentTime] = useState(0);

  useEffect(() => {
    fetch('http://localhost:8080/time').then(res => res.json()).then(data => {
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
### Step 2: Serve React-App application

Test everything is working by launching react app that we just created.
```
npm start
```
When you access `http://localhost:3000` on your browser, you will see the `time` value from the back-end on the React app. Once you refresh the website, we will see a new time value again.
> Make sure the back-end docker is still running to display the time correctly.

Once everything is working on the local side, we will deploy the code to Google Cloud.
Please proceed to the next tutorial [here]({% post_url 2021-02-16-deploy-react-and-flask-with-firebase-hosting-and-cloud-run-part-2%}) for Part 2.
# Reference
1. [Cloud Run Quick Start Guide](https://cloud.google.com/run/docs/quickstarts/build-and-deploy)
2. [Hosting Flask servers on Firebase from scratch](https://medium.com/firebase-developers/hosting-flask-servers-on-firebase-from-scratch-c97cfb204579)
