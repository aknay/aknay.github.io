---
layout: post
title: "How to deploy containerized Flask application to a Kubernetes with Kubeadm on Virtual Machines - Part 1"
date: 2023-03-17 00:00:00 +0800
excerpt_separator: <!-- excerpt -->
---
This is the first part of a series of hands-on tutorials for deploying containerized flask application to a
Kubernetes with Kubeadm on Virtual Machine.
<!-- excerpt -->
# Overview
In this tutorial we are going a create a Kubernetes cluster with 1 master and 1 worker using `Virtual Box`.


## 1. Install VirtualBox
Depending on your OS, the installation will be different. You can refer to the [offical link](https://www.virtualbox.org/wiki/Downloads) for the installation.

If you are on Ubuntu, you can use the following command to install `VirtualBox`.
```
sudo apt update
sudo apt install virtualbox 
```

## 2. Download Ubuntu ISO
We will be using `Ubuntu 22.04` Desktop version. You can download the `ISO` file from the [offical website](https://ubuntu.com/download/desktop). 


## 3. Create a Master Virtual Machine

### 3.1 Create a virtual machine
Let's first create a master virtual machine.
1. Click `New` on VirtualBox.
2. For `Name`, type `master` or  or anything you like.
3. For `Type`, choose `Linux`
4. For `Version`, choose `Ubuntu (64bit)` and click `Next`
5. For `Memory Size`, selecte at least `4GB` just to make it safer.
6. From now until `File location and size`, just choose default value and click `Next`.
7. At `File location and size`, you need to choose at least `20GB`. Then click `Create`.

### 3.2 Change CPU count
We need to change the CPU count to at least 2 as Kubenetes needs at least 2 CPU per node.
1. Once the `Machine` is created, click `Settings` while selecting the machine. 
2. Go to `System` > `Processor` and change the processor count to at least `2` CPU.

### 3.3 Load ISO file to Storage
We need to load ISO to the drive. 
1. From the `Settings` of the machine, Go to `Storage`. 
2. Then click `Empty` under `Controller: IDE`. 
3. Then `Optical Drive` shoud appers on the left side. 
4. Click on the `disk icon` and a `drop down menu` will appear. 
5. Select `Choose a disk file..` and load the Ubuntu ISO file that you download from the previous section.
6. Click `OK`.

### 3.4 Installation Ubuntu on Virtual Machine
1. Click `Start` while selecting the machine. 
2. Select `Try or Install Ubuntu` after the machine booted up.
3. Select `Install Ubuntu` and following the instruction.
4. Choose `Minimal Installation` during the `Updates and other software` to minimize the installation time.
5. Then click `Install Now` to install. Do not worry if you see `Erase disk and install Ubuntu`. It will only erase data from this virtual machine which we don't have anything on. Then again just follow the instruction. 
6. While at `Who are you?`, make sure you select `Your computer's name` that you can recognize. I choose `master-vbox` and `worker-vbox` for the master and the worker respectivily.
7. Pick your `username` and `password` you like. Then click `Ok` and finish the installation.

## 4. Crate a Worker Virtual Machine.
Just repeated a previous steps to crate a work virtual machine. Take note of the following while creating the worker machine.
1. Set storage Size to be at least `20GB`.
2. Set memory to be at least `2GB`.
3. Set at least `2 CPU` count.
4. Choose `computer name` which must be different from the master machine. 

## 5. Setup NAT Networks
We want to use `ssh` from our physical machine to two virtual machines. So we need to setup the network.

### 5.2 Create a NAT Network
1. From `VirtualBox`, go to `File` > `Preferences`.
2. Then go to `Network` and click a `+ network card` icon. 
3. Double click on it the network. Set the network you like. Mine is `K8sCluster` and click `OK`.


### 5.3 Connect virtual machines NAT Network 
1. Shutdown both virtual machines. 
2. Then click `Settings` of the machine.
3. Click `Network` 
4. Choose `Attached to` to `NAT Network`. Select the NAT network name you created previously. Then click `OK`. 
You need to connect both virtual machines to NAT Network.

### 5.1 Getting IP address of each virtual machine
1. Turn on both virtual machines.
2. Go into each virtual machine and open the terminal.
3. Type in the following command and take note of the IP address.
    ```
    ip addr show
    ```
We need IP addresses from both machines. 

### 5.2 Install SSH server
1. While we are the terminal, install `openssh server` using the following commmand for the both machines.

    ```
    sudo apt install openssh-server
    ```
2. Check the installation with the following command. 

    ```
    sudo apt install openssh-server
    ```
    You should see `active (running)`.

### 5.3 SSH Port Fowarding 
1. Go to the `NAT Network` dialog that you just created.
2. Click `Port Fowarding` button.
3. Add the following rules. You need to set the `Guest IP` as IP addresses that you just recorded from the previous section.

    | Name       | Protocol | Host IP   | Host Port | Guest IP  | Guest Port |
    |------------|----------|-----------|-----------|-----------|------------|
    | ssh master | TCP      | 127.0.0.1 | 2222      | 10.0.2.15 | 22         |
    | ssh worker | TCP      | 127.0.0.1 | 2223      | 10.0.2.4  | 22         |

4. After you set the values, click `OK`.

### 5.3 Test the SSH connection to Virtual Machines
Let's ssh to our virtual machines.
1. Make sure you already have ssh client installed on your physical machine. If not, use `sudo apt install openssh-client` to install.
2. From your physical machine's terminal, I use the following command to connect to our master. 
    
    ```
    ssh -p 2222 aknay@127.0.0.1
    ```

The command should be `ssh -p <HOST POST> <USERNAME>@127.0.0.1`.
After successful connection, you should see that your hostname is changed to the hostname that you connected.