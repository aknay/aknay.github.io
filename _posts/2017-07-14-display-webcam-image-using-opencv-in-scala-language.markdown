---
layout: post
title:  "Display WebCam Image Using OpenCV in Scala Language"
date:   2017-07-14 10:00:00 +0800
excerpt_separator: <!-- excerpt -->
---
In this short example, we will use Scala language to retrieve webcam image with OpenCV SBT library.
<!-- excerpt -->

#### **Prerequisites**
1. Please make sure both Scala and SBT are installed on your system. Please follow my tutorial post [here]({% post_url 2017-05-09-how-to-install-scala-and-sbt-in-raspberry-pi-3  %}){:target="_blank"} to install. This is applicable for both Raspberry Pi or any Linux system.

#### **Reference**
1. Library from [bytedeco](https://github.com/bytedeco/sbt-javacv){:target="_blank"}

#### **SBT Plugin**
I am using scala version `2.12.2`. I added the plugin in `project/plugins.sbt` as `addSbtPlugin("org.bytedeco" % "sbt-javacv" % "1.14")`. You can check the latest version in their release from [here](https://github.com/bytedeco/sbt-javacv/releases){:target="_blank"}.

#### **Test Environment**
I tested on Ubuntu 16.04.2 using a simple USB webcam. No driver installation.  

#### **Example**
Before we begin, I must say `thank you` to those who write OpenCV plugin for Scala language. We all know that writing code in Scala language is a joy. Let's begin.

This is just a short program just to prove that the library is easy to setup and get the OpenCV working in a short period of time. Once you run the following code, the JFrame will be prompted with an image from your webcam.  

{% highlight scala linenos %}
package example

import org.bytedeco.javacpp.opencv_core._
import org.bytedeco.javacv.FrameGrabber.ImageMode
import org.bytedeco.javacv.{CanvasFrame, Frame, OpenCVFrameGrabber}

object OpenCvExample extends App {
  val canvas = new CanvasFrame("Webcam")
  canvas.setDefaultCloseOperation(javax.swing.JFrame.EXIT_ON_CLOSE)
  val grabber = new OpenCVFrameGrabber(0)
  grabber.setBitsPerPixel(CV_8U)
  grabber.setImageMode(ImageMode.COLOR)
  grabber.start()
  val img: Frame = grabber.grab()

  canvas.showImage(img)
}
{% endhighlight %}