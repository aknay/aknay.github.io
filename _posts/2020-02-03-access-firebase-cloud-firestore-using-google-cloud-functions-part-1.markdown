---
layout: post
title:  "Access Firebase Cloud Firestore using Google Cloud Functions - Part 1"
date:   2020-02-03 10:00:00 +0800
excerpt_separator: <!-- excerpt -->
---
In this tutorial, we will setup gloud SDK and Firebase CLI at the local machine.  
<!-- excerpt -->

## **Series**
There are two parts to this series. Please read [here]({% post_url 2020-02-03-access-firebase-cloud-firestore-using-google-cloud-functions-part-2%}) for Part 2.

## **Prerequisites**

1. You need to know how to create a Firebase project; You can refer to the starter guide [here](https://firebase.google.com/docs/firestore/quickstart).
2. Familiar with JavaScript/TypeScript
3. Familiar with Python

## **Overview**
In this tutorial, we will set up `gcloud SDK` to access `Cloud Firestore` from our local machine. Otherwise, you will not be able to access Cloud Firestore.

<ol>
<li>
    We first download gcloud SDK: You will be asked a few questions during this installation. By pressing <code class="highlighter-rouge">Enter</code> as default for most of the questions.
  <pre class="prettyprint"><code class="language-sh">$ curl https://sdk.cloud.google.com | bash</code></pre>
</li>
<li>
    We need to restart the shell by typing the following command.
  <pre class="prettyprint"><code class="language-sh">$ exec -l $SHELL</code></pre>
</li>

<li>
<p>
    Before you initialize gcloud, make sure you had set up your Firebase project at <a href="https://console.firebase.google.com/">firebase console</a>. Now we initialize our gclould SDK. You will be asked to select your account and the project you just created at Firebase console.
</p>
  <pre class="prettyprint"><code class="language-sh">$ gcloud init</code></pre>
</li>
<li>
    We want to log in to gcloud; You will be asked to log in to your account with a new webpage after you type the following command at the shell.
  <pre class="prettyprint"><code class="language-sh">$ gcloud auth application-default login</code></pre>
</li>
</ol>


## **Setup Firebase CLI**
Now we want to write cloud function to serve between your Could Firestore and your API call from other machines. I strongly suggest you watch [this video series](https://firebase.google.com/docs/functions/video-series) from Google to familiarize yourself with  `TypeScript` and how the overall functions gonna look like.

Now, let's start setting this up.

<ol>
<li>
    We first just start by installing <code class="highlighter-rouge">npm</code>.
  <pre class="prettyprint"><code class="language-sh">$ sudo apt install nodejs</code></pre>
</li>
<li>
    You should check the installation of node and npm by.
  <pre class="prettyprint"><code class="language-sh">$ node --version</code></pre>
  <pre class="prettyprint"><code class="language-sh">$ npm --version</code></pre>
</li>

<li>
    We will now install <code class="highlighter-rouge">Firebase CLI</code>.
  <pre class="prettyprint"><code class="language-sh">$ sudo npm install -g firebase-tools</code></pre>
</li>
<li>
    We will create a folder for our project and then we go into the folder that you just created.
  <pre class="prettyprint"><code class="language-sh">$ mkdir firecast</code></pre>
  <pre class="prettyprint"><code class="language-sh">$ cd firecast</code></pre>
</li>

<li>
    We will log in to firebase at shell; you will be introduced with new pages if you haven't logged in yet.
  <pre class="prettyprint"><code class="language-sh">$ firebase login</code></pre>
</li>

<li>
    After authenticated, type in the following command to initialize. While you are running it, select <code class="highlighter-rouge">Functions: Configure and deploy Cloud Functions</code> using arrow key and space bar to select an item. Then select Firebase project you want to use. After that, Choose <code class="highlighter-rouge">TypeScript</code> as a language to use for this project. You want to choose <code class="highlighter-rouge">TSLint</code> to catch bugs and enforce style. Next, we select <code class="highlighter-rouge">Yes</code> to install dependencies with <code class="highlighter-rouge">npm</code>.
  <pre class="prettyprint"><code class="language-sh">$ firebase init</code></pre>
</li>

</ol>

## **Next**
We have set up `gcloud SDK` and `Firebase CLI`. Now we are ready to write some code. Let's proceed to [Part 2]({% post_url 2020-02-03-access-firebase-cloud-firestore-using-google-cloud-functions-part-2%}). 


