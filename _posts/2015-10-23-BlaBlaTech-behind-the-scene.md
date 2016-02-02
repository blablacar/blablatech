---
layout:     post
title:      "BlaBlaCar Tech, behind the scene"
tags:       [application-architecture, technology]
authors:    [francis-nappez, nicolas-blanc, nicolas-schwartz]
---

(*This article is an english translation of [this article](http://www.journaldunet.com/solutions/cloud-computing/1165036-coulisses-techniques-blablacar/) originally published on "Journal Du Net" website)*

**The ridesharing service created in 2006 has made major progress. The small, basic site developed in PHP has been replaced by an architecture that is able to meet growing demands on activity**. Francis Nappez, technical director at [BlaBlaCar](http://www.journaldunet.com/blablacar/), takes JDN behind the scenes of their rapid rise to success.

Created in 2006, BlaBlaCar is now a well-known service among ridesharing fans. Given today's environmental concerns and the financial difficulties many are facing in France, it’s a mode of transportation that is gaining in popularity.

The BlaBlaCar community has gone from 2 million to 20 million users in 3 years, proof that this service is still relevant - as if any proof was needed. The BlaBlaCar app has been downloaded over 5 million times.

## A highly available platform with agility constraints

![The co-founders of BlaBlaCar: Francis Nappez, CTO, Nicolas Brusson, COO, and Frédéric Mazzella, CEO. © BlaBlaCar](/images/2016-01-19-BlaBlaTech-behind-the-scene/cofounders.jpg)

With sites in 19 countries, mostly in Europe, but also in Russia, Mexico and now India, [BlaBlaCar succeeded in raising 200 million dollars in September 2015](http://www.journaldunet.com/web-tech/start-up/1161704-blablacar-annonce-une-levee-de-200-millions-de-dollars-et-devient-une-licorne/). Francis Nappez, co-founder and technical director of BlaBlaCar told us about the technical infrastructures behind this incredibly successful service. "We are currently on the third version of the BlaBlaCar technical platform. The first version, from 2008, was a very basic prototype" he explained. "From 2008 to 2013, we worked very hard to lay down a foundation for the future. Since 2014, we've been operating on the third version of the platform."

The technical platform was built with several goals in mind. First, the site needed to accommodate the increasing number of visitors. Over a million single users used the service every day, either via the Web or mobile version. But the platform also needed to be able to evolve quickly. "Today, our service is available in 19 countries, each one faced with different challenges," Francis Nappez said. "Our product must be able to adapt to the country where it is being used and have features that appear according to the region and depending on the maturity of its marketplace. We must have the capacity to deliver features for all the countries at once, a group of countries or one single country."

## Agile methods and continuous integration

With agile methods and a continuous integration design, BlaBlaCar is structured to maintain this flexibility even though the number of visitors has greatly increased. Francis Nappez gave us the keys to BlaBlaCar's success.

With a strong international presence, [BlaBlaCar](http://www.journaldunet.com/blablacar/) is supported by a network of 6 data centers on 3 continents. They are located in Singapore, Miami, London and Paris. The BlaBlaCar production platform is unique in that it combines traditional servers with virtual servers located on a private cloud and public cloud: a hybrid platform that currently includes 300 physical servers and 150 virtual machines (on the private cloud).

## A private cloud combined with Amazon Web Services

![Inside a BlaBlaCar data center. © BlaBlaCar](/images/2016-01-19-BlaBlaTech-behind-the-scene/datacenter-1.jpg)

BlaBlaCar uses services of the Amazon Web Services public cloud, among others, for storage. "It wouldn't be worth it to build our own storage infrastructure," Nappez said. "Amazon does a very good job already, so we use S3 to store member photos, for example. In parallel, we use EC2 for batch processing of large volumes of data." Less than 10 instances of EC2 run permanently on Amazon Web Services as the processing power of the public cloud is used mainly for occasional needs. "We use Amazon to manage traffic overflow and periodic calculations. So the Cloud allows us to perform the tasks our physical infrastructure can’t, like mobilizing 100 machines simultaneously to do a calculation and shutting them down immediately afterwards," he added.

VMware drives the private cloud

The majority of production is therefore ensured by the racks of servers BlaBlaCar deploys as needed, as well as by its private cloud, built on VMware technology. "The virtual servers provide us with agility for our deployments and allow us to implement services that do not monopolize an entire machine when they are not called on," Nappez concluded.

## Predictable load increases

Moving toward a 100% cloud-based architecture, like Netflix has done, is a question the technical director is still considering. "First, there's the question of internal skills and team experience. When we chose this hybrid architecture, it was because we had the skills," he said. "At BlaBlaCar, I think we master the cost and performance of our servers fairly well - whether we're talking about bare metal servers or virtual ones. Our nominal traffic is not really seasonal and it is fairly easy to project over a 6-month period." Scaling up the internal BlaBlaCar platform remains predictable and manageable for the startup. The peaks in traffic, about 20 to 25% during certain times of year, can be anticipated by acquiring additional server resources and bandwidth. Francis Nappez is already preparing the platform in order to anticipate Christmastime activity increases.

To go along with this load increase and manage the hybrid, and thus heterogeneous architecture, Nappez decided to automate administrative tasks and server provisioning. "Machines are now provisioned by scripts, rolling out strategies according to the roles of each server. We're capable of saying that a certain server, identified by its IP address, MAC address, etc., is to play a certain role in the architecture," he explained.  To manage deployments, the BlaBlaCar team made the decision to use Foreman solutions for provisioning and Chef for configuration management. "It may be a Web server with a relatively simple role, or it may be a database. In this case, there is a scenario for installing the machine, synchronizing the data, implementing monitoring, declaring availability for use by the application, etc." said Nappez.

## Automated deployment of virtual machines by roles

![Photo taken in BlaBlaCar data center. © BlaBlaCar](/images/2016-01-19-BlaBlaTech-behind-the-scene/cables.jpg)

With Foreman and Chef, BlaBlaCar was able to completely automate deployment of machines by roles, whether on hardware servers, virtual servers (on VMware) or instances of Amazon Web Services. "Chef is a repository, but also a way of working," the director stated. "Everyone knows very precisely what is on each machine. Everything is described in the scripts. The automated process is readable, transparent and everything is traceable. For an infrastructure team that now has fifteen members, it's very useful to have this common, solid code base that can be used to manage all the machines."

With his team of architects, Nappez also consolidates the platform. "What we want to do now is automate the installation of entire racks. Our growth is making it so that we need to deploy entire racks of servers." The objective? No longer have to travel on site to launch a new rack in production. This service is purchased from a local service provider who physically installs the rack in the data center. Then, the idea is that all server configurations and installations can be managed remotely, in an automated manner. "Once the rack is connected to the network, we want to be able to deploy all the servers according to their roles. To achieve this, we are currently adding Collins [from Tumblr] to pause all the new servers," Nappez explained.
CoreOS to support containers

Next, BlaBlaCar plans to use an even more generic OS, such as CoreOS, and to be able to deploy containers on it. "We want to be completely agnostic in terms of hardware and have an applicable procedure on both our cloud and AWS."

## Rocket to be deployed before the end of the year

For deployment of the applications themselves, the next iteration of the BlaBlaCar infrastructure will be based on containers. "This aligns with what we're trying to do in that it allows us to be resistant to hardware issues and to manage physical servers in Paris or Miami and instances of Amazon Web Services in Singapore and Sao Paolo. We'll choose the most obvious solution that is easiest to manage, depending on the geographical location," Nappez pointed out. "We are therefore going to deploy Rocket at our data center [the Docker fork developed by the CoreOS team] to be able to deploy a service, a database or web server container at any location in the same way."

The first step of this approach is to transform the BlaBlaCar applications so they’ll be more service-oriented than they already are. "We're constantly working to further break down our application. Then we want to deploy services in containers according to the machine resources required and the geographical position. We are in the process of deploying it at our Parisian data center and we'll have implemented Rocket before Christmas."

Historically, the [BlaBlaCar](http://www.journaldunet.com/blablacar/) site was developed using PHP. But today, the technical platform has diversified, using a lot of Java and Python to meet specific needs. "For certain backends, it is essential to be in Java. I'm thinking of the Cassandra database in particular. Java therefore occupies a significant place in our architecture, but for the time being, I'd say 20% of the code we produce is in Python, 10% in Java and 70% PHP," the CTO specified. In the PHP layer, which notably manages the frontend of the site, BlaBlaCar just migrated to Symfony 2 with the help of SensioLabs - the company founded by the PHP framework creator, Fabien Potencier.

## ElasticSearch engine under the hood

![Francis Nappez is the CTO of BlaBlaCar. © BlaBlaCar](/images/2016-01-19-BlaBlaTech-behind-the-scene/cto.jpg)

BlaBlaCar has implemented the high performance message broker RabbitMQ between the services developed in these various languages. "The idea is to unload the frontend servers as much as possible using an asynchronous task process. RabbitMQ plays the role of intermediary while all the processes are carried out, without slowing down the user experience," he explained.

In terms of databases, the majority of BlaBlaCar's data is stored on the MariaDB relational DBMS. Currently, the BlaBlaCar service is based on 3 MariaDB clusters. However, various systems are implemented in order to relieve the relational database and provide other data processing possibilities. "We've been using ElasticSearch for several years now. The first version we installed must have been 0.19. We started very early because we quickly saw that it could be a very innovative and promising solution" he confided. "We deployed the Solr search engine first, but some of the problems we encountered could not be resolved. That's when we switched to ElasticSearch. We definitely don't regret this choice because it's a tool we use all the time, both as the site search engine and for the backoffice."

## Redis for cache, Cassandra for email

Beyond these two key building blocks, the BlaBlaCar architects opted for Redis to ensure storage of key/value type data, the persistent cache or various counters; but also for Couchbase for all data related to sessions. It's a solution the technical director describes as having "very good availability and performance." The latest building block to make its appearance in the BlaBlaCar architecture is the NoSQL Cassandra database. "The entire messaging system we offer to our users operates on Cassandra," Nappez stated. "We observed that the volume of message exchanges was growing among members and we had to implement a solution that could handle scaling up from a technical point of view."

[BlaBlaCar](http://www.journaldunet.com/blablacar/) relies on a team of about 40 developers and 15 architects. The architects are committed to building the platform while the developers work to enrich the features and internal tools of the service. "We have almost as many internal tools as those we make available to our members," he concluded. "Managing 20 million people requires a significant backoffice and we're building that backoffice ourselves."

## A Big Data platform on Vertica's Hadoop

![BlaBlaCar data center. © BlaBlaCar](/images/2016-01-19-BlaBlaTech-behind-the-scene/datacenter-2.jpg)

In addition to the architects on the one hand and the developers on the other, BlaBlaCar relies on a third, data-oriented team. Their role is to provide raw data or prepared reports to other teams, be it to present statistics on the community or press, or to improve the efficiency of marketing campaigns. "We rely on Vertica's Hadoop stack for the processing part and on Tableau Software for analysis and report creation. We don't really have any true Data Scientists," he revealed. "But rather what I call Data Engineers who provide data, according to our computing capacities, to the marketing and communication teams." A Hadoop cluster operates continuously to collect all logs issued by the production servers. The largest calculation batches are launched on Amazon Web Services.

## Up to 10 deployments per day

The continuous integration process has become the main force behind the site's growth. BlaBlaCar currently performs up to 10 deployments per day, including minor corrections and sometimes new features for a certain geographical zone, for instance. "A 'slow' Monday would be a day with just 2 deployments!" Nappez joked. "The technical aspect is the first hurtle to achieving such results. We have to be able to deliver deployments automatically and have extremely reliable testing processes in order to feel serene when we do deploy. We're required to improve this process continuously. We use the Bamboo integration solution from the [Atlassian](http://www.journaldunet.com/solutions/cloud-computing/1168969-atlassian-l-editeur-qui-fait-un-carton-aupres-des-equipes-agiles/) suite for builds, unit tests, etc." More than 10,000 unit tests are performed before each deployment. This is how BlaBlaCar ensures that a correction or new feature will not cause side effects or regression. A new version build can be achieved in 15 minutes.

All BlaBlaCar developers manage their deployments themselves.

"The other issue is team involvement. What matters most to me is that the person in charge of a development project, who holds the keys to a new feature of our service, can see it through, all the way to deployment." This means that as early as their first week on the team, BlaBlaCar developers will perform their first deployments. "We want to demystify this step. Deployment is a part of the job. It's a normal step in the process and so they should get used to it starting from their first week with us. And it's only natural that with a team of 50 people, we manage to achieve up to a dozen deployments per day. We want our platform to be as agile as possible."

## Experienced Scrum users, BlaBlaCar is now leaning more toward Kanban

In terms of methodology, BlaBlaCar has made widespread use of agile methods; Scrum in particular. However, Kanban, a slightly different approach, is used simultaneously and has been getting increasing attention from teams. "We try to leave the door open to common sense when needed, so we can get the best of both worlds. We want to be very structured with the ability to build projects among several teams, but also capable of reacting quickly to manage any problems, especially when facing production contingencies." Francis Nappez noted that, like all Internet services, their activity ascribes to a "production first" philosophy. If a problem appears in production, all teams must be able to react quickly. Indeed, what's the use of agile methods if they don't increase flexibility?
