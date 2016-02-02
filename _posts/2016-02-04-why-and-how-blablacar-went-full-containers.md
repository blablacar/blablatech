---
layout:     post
title:      "Why and how BlaBlaCar went full containers"
tags:       [containers, architecture]
authors:    [simon-lallemand]
description: Why and how BlaBlaCar went full containers
---

## Enter the hardware industrialisation era
At BlaBlaCar, we’ve always run our software on bare-metal servers. 
There are several reasons for that:

 * We’ve always had the skills in the team to manage the hardware and the network
 * We can optimize the performance from end to end, particularly the network
 * We don’t want to be bound to a cloud provider
 * Multi-cloud provider is a dream

Until 2 years ago, we used to buy different servers for each service that we needed to run according to the requirements in terms of RAM, CPU and disks of those services. But as our business is growing very fast, that approach was not sustainable anymore, and we couldn’t spend enough time to deal with too many types of hardware.

So we decided to embrace industrialisation of the hardware by choosing 2 or 3 server configurations.
This has brought us two things: 

 * lower prices for buying in bulk the same server of reference
 * less time to integrate the servers into the platform.

So everything was great…!
 
But then we had another problem to solve: even if our new servers are the same, our services are still very heterogeneous and we need to deploy them in the most efficient way.

## Resource allocation
For quite some time we’ve been using Chef extensively for all our services at BlaBlaCar. All our team was using it and we had a good workflow with code review, testing...

Unfortunately, even if Chef can deploy multiple services on one server, it doesn’t provide isolation, so you end up with implicit dependencies.

Isolating multiple services on one server is not new; we already had a small VMware cluster for some services that didn’t need much performance, such as the ones for tests, pre-production, infrastructure…

We didn’t want to continue using VMware because it was not that easy to automate, not as performant as baremetal and too expensive when you have lots of nodes. So, instead of going with another virtualisation solution, we decided to make the big jump, straight to the containers.

## Deployment speed
Nowadays in the physical world, launching new resources is slow. Launching new virtual resources in VMware is faster, but still slow. You know the drill. Your development team needs to access your resources in seconds. Moving to containers was going to give us access to that kind of lightspeed deployments.

## Do you docker?
No we’re not using Docker. Even if we started considering it at the beginning of our tests, we quickly realised that the limitations of the product, at that time, were too important for us to go in production with it. 

When we were still experimenting with containers, the guys from CoreOS announced a new runtime: [rkt](https://github.com/coreos/rkt) (or Rocket as it was still called in early 2015). We were already convinced by the OS they made for the simplicity of administration, the security and the atomic updates and we gave a try to that new tool. We were surprised by the stability of rkt even in the very early versions, and all the important features we wanted were added quickly by the CoreOS team. We became confident that rkt would keep improving and, as its architecture seemed better for production usage, we decided to build our new platform with it.

## Building the containers
Actually, running services in containers is not very difficult. The hard part is making a build process that can scale to hundreds of containers with the 15+ engineers of the architecture team working on it at the same time. In other words, we needed to develop standards.

You can’t just say "everyone writes their own shell scripts"--unless you want your platform to end up in complete chaos.
We had the same problem when we started using configuration management with Chef, and after several iterations we came up with internal policies on how to build a cookbook, a role cookbook… 

Our first approach was therefore to build the container images with special Chef cookbooks and packer. Once the workflow was ready, we made a workshop with the whole team to try it, and we quickly realized that it was cumbersome to use chef for such a task. The main drawbacks were: 

 * It was complex to write cookbooks only to install some packages and some files.
 * It was slow to converge from scratch each time.
 * We could not easily customise the config at the start of the container (when you need to change the ID of the node in a cluster, for instance).

So we went back to the whiteboard to find a solution that would address those issues. Our requirements were:

 * quick build (you don’t want to wait 10min for a build to finish)
 * easy to understand for newcomers (You don’t want to spend two weeks explaining your process)
 * as little code replication as possible (For the same reason that in Chef we have role cookbooks that depend on app cookbooks) 
 * templating that could be resolved at the start of the container (binary immutability is good, but configuration changes)
 * a good integration with rkt (we want to create ACIs and PODs that can directly be used with rkt)

## CNT
As we couldn’t find a tool to do all that, we started to build our own. At first [CNT](https://github.com/blablacar/cnt) was written in bash but as the complexity was growing, we decided to rewrite it in go.

CNT is a command line utility designed to build and configure at runtime App Containers Images (ACI) and pods based on convention instead of configuration.

We think that CNT deserves a dedicated blog article, stay tuned…

## The (Core)OS
Now that we were able to build all the containers we needed, we had to decide the OS to install on our servers. We had been testing [CoreOS](https://coreos.com) for a while during our exploration phase, and we liked its simplicity, its security and the fact that it could only run containers. That last point was very useful to enforce our full container policy; we wanted a complete migration in order to have only one way of doing things.

CoreOS was easy to install on baremetal by following the documentation which is complete and up to date. We are still running Chef on it to configure some system services but we are considering alternatives.

## Orchestration
We took a bottom-up approach on this project, so logically, the orchestration problem came last. At that moment we had a time constraint that didn’t allow us to test and deploy a full orchestration solution.

We decided then to go for a lighter solution that comes by default with CoreOS: fleet. 

But fleet alone wasn’t enough to run a lot of production services, we didn’t want to write and submit by hand all the systemd units. That’s why we wrote a tool for that: [GGN](https://github.com/blablacar/ggn) (aka. green-garden)

GGN generates, starts and updates systemd units for all our services using templates and attributes per environment, per service and per POD.

We will give you more details on these tools and how they work in a future article.

## Conclusion
 * It took us 7 months to go from the idea of using containers to a state where more than 90% of our prod services are running in containers.
 * We have dev, system and network skills gathered in the same team, and it really helped to go through this big change so quickly.
 * We made LOTS of iterations, adapting the process with the feedback of all the people in the team until it was easy enough to use.
 * We really wanted to keep Chef as we have invested a lot of time on it, but we learned the hard way that classic configuration management tools are not fit to build and manage containers.
 * We are very happy to have built our platform with rkt. The design proved to be very fit for production and [now that it reaches v1.0](https://coreos.com/blog/rkt-hits-1.0.html), you have no reason not to give it a try.
 * The move to containers brought us a huge gain in terms of delivery of new services and scaling of existing ones. That was expected but still, it’s very sweet.
 * We still have a lot of work ahead, especially with orchestration, automating more stuff with continuous integration and improve our tools. If you’re interested in working on such projects : [we’re hiring](https://blablacar.com/dreamjobs) !


*Thanks to Arnaud, Rémi, Jardin, Julien, Bob, Usman, Max and MattKetmo for reviewing this article*
