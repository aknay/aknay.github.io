---
layout: post
title:  "Install Golang on Ubuntu using Snap"
date:   2021-12-28 21:00:00 +0800
excerpt_separator: <!-- excerpt -->
---

This is a short tutorial to install Golang on Ubuntu using Snap. 

<!-- excerpt -->

# Overview
In this tutorial, we will first install Golang on Ubuntu using Snap. Then we will test our setup using a simple program.

# Prerequisites
1. Ubuntu
2. Snap

# Installation
Let's install Golang first.
```bash
sudo snap install go –classic 
```
Once the installation is done, we can check the installation with the following command.
```bash
go version 
```
You will see something like this on your terminal `go version go1.17.5 linux/amd64`

# Dependency
In order to install other libraries and to keep track of your dependencies, you need to create a `mod` file.
First, let's create a project folder.
```bash
mkdir MyGoProject
cd MyGoProject
```
Inside the folder, just run the following command on the terminal. Here, we are creating `MyFirstProject` as a module. For better module naming, you can follow [this link](https://go.dev/doc/modules/managing-dependencies#naming_module) from Golang Website.

```
go mod init MyFirstProject
```
This will create a `go.mod` file inside the folder. If you open it, you will see your module name as `MyFirstProject`.

# Code
Let's write our first hello world program. Inside the same folder, create a file name called `main.go` and paste the following code.

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

After you save the file, run the following command.
```bash
go run main.go
```

You will see our hello world message as `Hello, World!`
> Side Note: If you are using vscode to edit, make sure you add the following setting in your vscode. Refer to [this link](https://github.com/golang/vscode-go/issues/1411) for more info.
```json
   "go.alternateTools": { 
        "go": "/snap/go/current/bin/go" 
    },   
```


# Reference
1. [Tutorial: Get started with Go](https://go.dev/doc/tutorial/getting-started)
