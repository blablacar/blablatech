---
layout:         post
title:          Android development at BlaBlaCar today
tags:           technology
authors:        [alexandra-tritz]
description:    In this article, we are going to talk about the mobile strategy of BlaBlaCar, more on Android side, give you a feedback from our experience and talk about what we will do in the future.
---

## Overview
In this article, we are going to talk about the mobile strategy of BlaBlaCar, more on Android side, give you a feedback from our experience and talk about what we will do in the future.

This is  a talk that we gave during a [PAUG](http://www.paug.fr/) meetup in march 2015.

** What is PAUG? **

PAUG stands for **P**aris **A**ndroid **U**ser **G**roup. It can be defined as "The community of Android professionals and enthusiasts in Paris".

They are organising a [meetup](http://www.meetup.com/Android-Paris/) each month to talk about the latest tools and innovations about Android. They also organized the [“Droidcon”](http://fr.droidcon.com/2014/) event in Paris, a two-days conference centered around Android technology.

<p class="text-center">
    <img src="../../images/2015-03-30-android-development-at-blablacar/blablacar-droidcon.jpg" alt="Droidcon Paris" width="400px" />
</p>

You can find the video of our talk [here](https://www.youtube.com/watch?v=B7jvwUbqApc) and the slides [here](https://speakerdeck.com/alexandratritz/developper-une-app-android-en-2015).


## Become mobile first

As the mobile market share keeps on growing, BlaBlaCar strategy is to become a “mobile first” company. It means that we really want to target mobile users. Today, BlaBlaCar has launched its app in 14 countries, and the aim is to keep on going.

![](/images/2015-03-30-android-development-at-blablacar/worldwide-expansion.png =600x)

<br>
To give you an example, the percentage of mobile usage two years ago was only 10% in the whole BlaBlaCar experience and today, it's more than 50%.

![](/images/2015-03-30-android-development-at-blablacar/graph-mobile.png =600x)

This is a real challenge as we also want to expand internationally, and this brings its own set of constraints.

## The existing

We started the development of the new android app internally at BlaBlaCar in 2013. Back then, choices were made, it’s what we call the existing.

#### Librairies
We are using some libraries in our Android app, I will go through them quickly.

* **[Butterknife](http://jakewharton.github.io/butterknife/)**, view “injection” library for android
* **[UIL](https://github.com/nostra13/Android-Universal-Image-Loader)**, for loading, caching and displaying images on Android
* **[Retrofit](http://square.github.io/retrofit/)**, a type-safe REST client for Android and Java
* **[Otto](http://square.github.io/otto/)** event bus library for Android
<br><br>

#### Translations
Translations are always a huge step when working on a mobile app, especially if you make it available in 14 countries like we do.
When developing the app, we had some major constraints about translations: we had to be able to update the translations often, **without updating the app in the store**. We also had to find a tool that could be easy to use for professional translators, as we developers do not have time to change the translations.

Our tool is called “OpenLocalization”. It’s an open-source project that has been developed by [Matthieu Moquet](/authors/#author-matthieu-moquet), one of our Web Engineers here at BlaBlaCar. You can check the project here: [http://openl10n.io/](http://openl10n.io/)

![](/images/2015-03-30-android-development-at-blablacar/openl10n.png =600x)

But how does these translations get deployed? It’s really simple. First, the developers create keys for the translations they need. Then, they push these keys onto openl10n, where they are fully accessible for the professional translators. When the translations are done, they are pushed onto Amazon S3 and the translations in the app are automatically updated at every launch.


## What we are currently working on

As the mobile technology keeps growing and expanding, we have to quickly adapt to mobile evolution. We have to think our app to fit the current standards and this is part of what we are doing today.

#### Robospice / RetroFit
What is [Robospice](https://github.com/stephanenicolas/robospice)? It's the library we use in our app to handle asynchronous network requests. It supports REST and caches results, which is a benefit for us when making requests from countries that are far away. 
The added value is that it can be coupled with RetroFit very easily. You can see in the [slides](https://speakerdeck.com/alexandratritz/developper-une-app-android-en-2015) how you can implement it in your own app.

#### API Optimisation
Our app is really depending on our API. Because we are launching countries that are far away, we need to anticipate any problem. Also to improve the overall fluidity, we have decided to change the way we get the acess token for any mobile user.

This is what we had before:
![](/images/2015-03-30-android-development-at-blablacar/API-1.png =600x)

You can see that this process is pretty tedious. The mobile application will ask our server for a token, which will be checked by MySQL. If this token has the necessary rights, it can make calls to our main Symfony application. Anytime we wanted to check, we had to go through all this process. And the main drawback is that our servers are located in Paris, so if the call comes from India for instance, it can take a lot of time, which is not user-friendly.

We decided to cut the link between OAuth2 and MySQL:
![](/images/2015-03-30-android-development-at-blablacar/API-2.png =600x)
<br><br>
This is what we have now. We are using [JWT](http://jwt.io/): a signed stateless token that allows us to put all the information that the authenticator needs.
![](/images/2015-03-30-android-development-at-blablacar/API-3.png =600x)

This environment needs to be deployed as close as possible to the user, which aims to reduce the latency without jeopardizing the security.
<br><br>

#### Monitoring
We did set up some monitoring in our apps. It is useful for us because it can help us on many points. First, it allows us to mesure the impact of the optimisations that we do. It can help us detect problems and trigger alerts before any critical problem.
It also came with constraints: we had to be able to configure, enable and disable it over the air.

For that we used a custom solution: we send those logs to [Logmatic](http://logmatic.io/) by hand. We wait to bulk on 10 requests and flush them when we are done.

![](/images/2015-03-30-android-development-at-blablacar/logmatic.png =600x)

This system allows us to follow in real-time the average response time of all requests we send in order to detect any problem.
<br><br>

#### Crashlytics

Crashes in an Android app should not be taken lightly. That’s why we are using the tool provided by Google, the [Google play console](https://play.google.com/apps/publish). However, you have to know that the crashes that you can see on these reports are not all the crashes that happen in real life to all of your users. Indeed, **the number of crashes that you can see are only the crashes reported by the user**, when he clicks “submit” in the popup happening after a crash. If he dismisses it, the crash will never be reported and you will never know. That’s why we are using another tool, called [Crashlytics](https://try.crashlytics.com/). It’s part of the Fabric suite, made by Twitter. It’s a really powerful tool, because all the crashes are reported here. Its added value is also that you can follow the number of users currently in your app and the percentage of crash-free users. The crashes are also automatically ordered by gravity called “levels” so you can know in a glance which ones needs to be treated in priority.
<br><br>

#### Google+ community 

We did create a Google+ community back in the time when we were switching from our old app to the new one. We wanted to have more feedback and interactions from our users. We have about 150 members in this community which are very active. They submit new ideas of features or report bugs several times a week. 


## What about the future?

For the future, we want to work on different axis.
First, we are integrating the latest version of [Appcompat](https://developer.android.com/tools/support-library/features.html) to keep us up to date with the latest features while having a retrocompatibility with the older versions.

We also want to optimize our continuous integration, because it’s the main tool that will ensure that we don’t have any regression.

We are also working on integrating the material design into our app, so it can be prettier and up to date with the latest guidelines.

And last but not least, we want to add more features for wearable products.

