---
layout:         post
title:          Swift Meetup Paris recap
tags:           [conferences, mobile]
authors:        [mickael-rodrigues]
description:    The recap of the Swift Meetup Paris talk "Unit tests with Swift" of Tuesday 26 May by Mickael Rodrigues.
---

After frequent exchanges with the Swift Meetup Paris organisers, BlaBlaCar finally hosted one of their meetups on Tuesday 26 May. An interesting topic emerged from discussions with another speaker of this meetup: unit tests on Swift. We decided to split the talk in two parts:

1. the first that focuses on the theory and some live coding
2. the second giving feedback from an existing project

This blog article will explain you my part, the theory of unit tests with Swift. 
First of all, go back to Objective-C :)

## Objective-C

In Objective-C you have many methods to do testing. You can use some frameworks like OCMock, GUnit, OCUnit, etc. Or you can use method swizzling:

<p class="text-center">
    <img src="../../images/2015-05-29_swift-meetup-paris-recap/method-swizzling.png" alt="Method swizzling" />
</p>

The main goal of this method is to change on runtime the implementation of your method. In this case, we want to test if the user is connected. We actually don’t care about how the “connected” property is set (downloaded via the cloud, the user default, etc.), we just need to test the case when the “connected” property returns “YES”. We will use method swizzling to do that on runtime. We keep the reference of “connected”, we replace it by “fakeConnected” and do the test. This is simple in Objective-C, in Swift this a little trickier.

## Swift

Method swizzling just doesn’t exist in Swift. We had to use other Swift methods like polymorphism or the closure. I will explain this 2 methods below.

### Polymorphism with Swift

The goal of this method is to make a class believe that it is calling a particular class, while it really calls a different. Here is an example of this method:

<p class="text-center">
    <img src="../../images/2015-05-29_swift-meetup-paris-recap/swift-polymorphism.png" alt="Swift Polymorphism" />
</p>

For this example we want to test that the “alertView” is displayed when the “ViewController” is loaded. 
On the application side, you have a public property “alertView” on “ViewController” with type “UIAlertView”. This property represent the alertView to be displayed. In the “viewDidLoad”, we call the method “show” of the “alertView”.
On the test side, we will create a “FakeAlertView” class that inherits from “UIAlertView”. This class will replace the existing “alertView” public property. This will give you the ability to override the “show” method and write your own code to test (where we set “showWasCalled” to true).
Finally, on the test side, you call “viewDidLoad” of a “ViewController” instance and check if “showWasCalled” is true. This will test whether an “alertView” is displayed when the “ViewController” is loaded.

### Closure with Swift

Same goal as for the Polymorphism example, making a class believe that a particular class is called, while another gets called. Here is an example of this method:

<p class="text-center">
    <img src="../../images/2015-05-29_swift-meetup-paris-recap/swift-closure.png" alt="Swift Closure" />
</p>

For this example we want to test that the “alertView” is displayed when the “ViewController” is loaded.
On the application side, you have a private property “alertView” on “ViewController” with type “UIAlertView” and a public property “showMethod” which is a function. “alertView” represents the alert to be displayed and “showMethod” represents the method “show” of the “UIAlertView” class. First in the “ViewController” constructor, we assign the “show” method of “alertView” to “showMethod”. This will lead to have a function pointer that points to the “show” method. In the “viewDidLoad”, we call the method “showMethod” which is the “show” method of “alertView”.
On the test side, we will create a “FakeClass” with a function. This function will have the same signature of “show” method of “UIAlertView” class and will be used to affect “showMethod”. This will give you the ability to write your own code to test (where we set “methodCalled” to true).
Finally, on the test side, you call “viewDidLoad” of a “ViewController” instance and check if “methodCalled” is true. This will test that something (in our case an “UIAlertView”) is displayed when the “ViewController” is loaded.

You can see the entire Meetup here, my unit tests on Swift start at 46 minutes and 46 seconds:

<p class="text-center">
    <iframe class="youtube" width="560" height="315" src="https://www.youtube.com/embed/bOECDI6lD4k" frameborder="0" allowfullscreen></iframe>
</p>