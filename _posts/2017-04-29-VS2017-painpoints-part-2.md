---
layout: post
title: Visual Studio 2017 and the pain it brings part 2
date:   2017-04-29
categories:
  - VS2017
  - C++
  - Development
---

This will be a very negative post so I want to start off with saying that I
love Visual Studio, it's my favorite IDE for C++ and C# development. With that
said the upgrade to VS2017 has been less than smooth.

### Installer ###
Lets try to set up Visual Studio for C++ Universal Windows Platform development.

First we select the workload that looks like what we want

![Workload selection](/images/installer.png)

Next we make sure to check the individual components that seems related to what we need

![Component selection 1](/images/installer2.png)

![Component selection 2](/images/installer3.png)

Now we should be good to go, let's open Visual Studio and try to create a new project

![Project creation](/images/vs-missing-component.png)

Well that didn't go as expected. You would think that installing the workload `Universal Windows Platform Development`
would install the bits needed to actually create a project.

Right, what happens if we install the suggested bits? Apparently it suggest installing the ARM tooling.

![Component selection ARM](/images/installer4.png)

Can we create a project now? Lets try it out and see what happens

![Project creation success](/images/vs-create-project.png)

Success! But why oh why did this not just work from the beginning?
I picked a workload and it seems quite reasonable to expect that I can actually use that workload once it's installed.