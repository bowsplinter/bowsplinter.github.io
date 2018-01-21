---
title: "Bringing Technology to Construction"
author: "Vivek Kalyan"
date: "11/08/2017"
---

I spent the past weekend participating at Hood Disrupt 2017, a property technology hackathon. It was an extremely tiring hackathon, with things not working till 7am on judging day. But I am pleased to say that my team emerged with the grand prize of $5000!

<img class="center-block" style="padding: 10px 30px" src="/img/hd17-win.png">

## Problem

We were working on increasing the productivity of the construction industry (productivity has actually dropped over the past 50 years - which is shocking considering the productivity increase in other industries)

When we talked to experts in the construction industry, the one thing that always comes back is the lack of real-time communication. Coupled with the fact that construction companies can lose up to hundreds of thousands of dollars every day the project is delayed, it becomes imperative to improve their communication process.

## Solution

**Autonomous Spatial Mapping**

<img style="padding: 10px 30px" src="/img/spatialmapping.gif">

Using a stereo cam, with a built in IMU (inertial measurement unit) together with a roomba we had a robot that was able to do autonomous spatial mapping. The roomba would navigate around the room, while the stereo cam is used to map out the space in 3D. The computer attached constructs the model, and sends it to the browser.

**Real time in Browser**

<img style="padding: 10px 30px" src="/img/browser.gif">

With the 3D model, the space can now be viewed in the browser. It can be explored, using normal mouse and movement keys. Issues can be seen and highlighted using the issue management system in the browser. The issues can be tagged with different problems or users, and multiple stakeholders can communicate in real time over problems in the space. The application is built using **React** and **ThreeJS** (a library to work with WebGL to manipulate 3D graphics).

**Real time in Virtual Reality**

<img style="padding: 0px 30px" src="/img/vr.gif">

The model is also imported into VR. A user can immerse himself in the 3D space and can look around as though he is physically at the site. Markers can also be placed to denote issues. There is no way to describe and comment on issues due to the poor user experience of typing text in VR. However, with improvements to speech-to-text technologies, having a microphone for audio could be something to consider in the future. The VR application was built using **Unity**.

## Benefits

Having a real-time site model gives the ability to view the model offsite, view deviations from the blueprints and fix them immediately. There is also automated documentation - history of the model throughout the span of the construction is recorded. This allows better reviews of processes which help to improve learning. It also serves as a reference for future use, taking the guesswork about pillars, pipes, wires etc. during renovations and repairs.

The centralised issue management system serves a single platform to track all the issues and resolutions of the site. With the documentation, it increases collaboration as all parties can see what solutions have been attempted and the status of it.


## Not done yet

After deploying our solution, we now have access to a huge amount of labelled data. We have models, we have areas in this models where there have been issues, and we have the models after these issues have been resolved. This amount of clearly labeled data is the dream of anyone doing supervised machine learning.


This allows us to use artificial intelligence to learn common problems in building and automatically detect issues in the building. It could even suggest solutions based on the common resolutions of other projects. This will cause the construction industry to grow exponentially as now you are able to learn from everyone's past mistakes and also their solutions.

***

**Update: 05/10/2017**

featured on [Straits Times](http://www.straitstimes.com/business/property/proptech-takes-buildings-beyond-bricks-and-mortar)!

**Update: 21/01/2018**

<img style="padding: 10px 30px" src="/img/bcatalk.jpeg">

I was invited to give a talk about the hackathon and how technology can be applied to architecture and construction industry by BCA Singapore. While preparing for the talk, I realised the importance of cross industry experience. The idea of having a centralised issue management system was something that we borrowed from Github and their issues system. Due to our experience, it was unfathomable for us to imagine working without such a system.

Therefore, my takeaway for the talk was to keep to date with innovations in other industries and look for opportunities to adapt them and bring them to your industry.
