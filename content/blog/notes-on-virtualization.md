+++
title = "Notes on Virtualization"
date = 2024-01-11
+++

Hi there\! This is aimed to be a comprehensive guide to virtualization.

~Don’t know what virtualization is? Don’t worry you’ve come to the right place. Lol not at all~

“Virtualization” is a term to describe the virtual version of a machine

Let’s start at the beginning, you have an operating system running on some hardware. It’s a piece of software interacting with the hardware and abstracting away the hardware from you. 

You ask the question - Why can’t I run an operating system on top of my operating system? You can, you write software which emulates your hardware, ie, software which talks like your hardware does, now you can put an OS on top of that emulator.

This in effect is a virtual machine.

Now the host OS is still performing all of it’s functions and your hardware resources are divided among the different processes/applications, one of them being your VM.

So can you imagine that the VM OS either needs to be lighter than the host OS to function at the same speed.

This is fine. Why do you think we did this in the first place?

- For kicks
- If you had big hardware and you wanted to give people access to compute over the cloud you could run multiple VMs on your hardware with constraints of how much of the host resources can they use.
- Any application you run on your computer has a lot of “dependencies”, say you make a simple python application. The python version you’re using is one of the dependency. Python uses a lot of little libraries from your OS to work. These are all dependencies. So if you had to put your application up on someone else’s computer you would have to install all these dependencies there as well. What if instead you could just put your entire OS on their machine.

So what we do is put up the most stripped down version of our OS that can still run our application, on the the computer running all day \(cloud\).

These still take time to start up tho.

\[WHY?\]

So we made “containers” these are lightweight containers.

They spin up quickly and provide the isolation we wanted from VMs.

But, as I recently found out they are apparently not as isolated from the host OS as we would want from a security pov.

Enter microVMs.

The best of both worlds.

Lightweight, so they spin up fast.

Provide more isolation than containers.

From my observation it also seems that they have persistent state and containers do not because you create and use a filesystem file to start a microvm.

Although I do feel like the same can be achieved with containers by mounting that filesystem to the container. I’ve not seen examples of this because most people using containers don’t really need a whole filesystem attached to their container.

Whereas VMs which are typically used when providing compute access to someone also give them a filesystem.

So now we come to the more interesting part which is the new type of virtualization.

These might not strictly fit into what we think of as virtualization but they help us achieve the same goal - creating a machine agnostic isolated environment to run your code.

**Browser**

Your browser contains an engine/program_ _which runs javascript in a way that it does not have to interact with your host OS \(_isolated environment_\).

This website’s developer wrote js code which could run on that engine and now the same code runs on any computer that opens this website \(_machine agnostic_\).

With the advent of WebGPU, ie an engine in your browser which can run code on your GPU, many applications will perform some computation on your GPU before sending data to a server. This saves their bill and reduces your latency if done right. Win-win\!

An example of this is the famous dingboard which as yacine described, pushes some layers of the architecture used for segmentation on to the client.

**Workers**

Cloudfare workers are another example of this.

This is not as machine agnostic as writing js for browsers as all browser js engines can run the same js code but the code written for a cloudfare worker runs only on a cloudfare worker.

But this becomes less true since they have open sourced their worker runtime.

Another benefit of virtualization is that it aids scaling. If you had an application running on a host OS and you wanted to scale vertically ie running the same application on better hardware you would have to start another machine with better hardware and move the OS and app there, whereas when your app sits in a vm/container you can just tweak the resources available to that container or quickly spin up another vm with the same resources \(horizontal scaling\).

Cloudfare workers retain this benefit, in fact the whole point of using workers is to be able to deploy your app and forget about scaling, cloudfare takes care of both spinning up geographically spread out workers and allocating the resources to them.

