---
layout:         post
title:          🚀 Launch of The BlaBlaCar Developer Portal!
tags:           [API, developer, hacking]
authors:        [erwann-robin]
description:    In this article, we are going to introduce the new BlaBlaCar Developer portal and give you some tips about how it works.
---

First, we wish you an happy new coding year!

For this new year, we are proud and very excited to launch our API Developer Portal: [https://dev.blablacar.com](https://dev.blablacar.com)

 ![screenshot](/images/2017-01-30_BlaBlaCar-developer-portal/DevBlaBlaCarHomePage.png)

This portal will help everybody to build innovative mobility solutions, that feature carpooling trips.

It gathers all documentations and guides about integrating with BlaBlaCar, to easily connect with developers and partners and to help develop carpooling. It's the privileged way to connect to the BlaBlaCar's API, which gives access to the trip inventory of BlaBlaCar.

Discover on it how the BlaBlaCar API can easily enhance your app's user experience and take your innovation further with a wide range of new capabilities.

# Details

BlaBlaCar's API is a standard RESTful API, resources-based (even if we have extended the concept to provide an experience view of resources), with a bit of HATEOAS, and a touch of magic. We support both XML and JSON format, JSON being the default one.

The integration process is very simple:

1. Open an account on [dev.blablacar.com](https://dev.blablacar.com)
2. Create your API key on the [dashboard](https://dev.blablacar.com/developers/dashboard)
3. Test it live on the [API Explorer](https://dev.blablacar.com/api-explorer/)
4. Integrate it right away in your project using your favorite language / tool / libraries
5. 🎉

## Quota
New accounts start with a small quota to be able to do some queries right away. Bigger quotas are attributed on demand after a proven successful integration.

## What can be done?
The API gives access to our inventory of trips with free seats. You can request for a specific axis on a given date, and add some criterias to narrow the results to your expectations. For example, you can request for trips with more than 1 seat with the *"seats="* parameter, or you can ask for all trips which start from Paris next Friday evening.

## How to make queries?
You will need to send a GET query to:
> https://public-api.blablacar.com/api/v2/trips

and add your queries parameters.

All available parameters are explained on the [developer portal](https://dev.blablacar.com/docs/versions/1.0/resources/trips).

## What's behind?
We are using the awesome open-source API gateway: [Kong](https://github.com/Mashape/kong) to serve public-api.blablacar.com and to isolate our backend servers. We plugged it to [Cassandra DB](http://cassandra.apache.org), for scalability.
The portal itself is hosted by [Gelato](https://gelato.io), another great [Mashape](https://mashape.com) tool. Gelato maintain the list of API consumers on our Kong cluster, and their associated API keys (see: [Using Gelato with Kong](https://docs.gelato.io/guides/using-gelato-with-kong)).

We also uses the open-source [kong-dashboard](https://github.com/PGBI/kong-dashboard) tool, to facilitate the administration of Kong, and our usual stack of Grafana / Kibana to monitor the cluster health and follow the http logs.

This configuration allows us to scale and to maintain an updated list of consumers while behind efficient in developing new features.


We’d love to work with developers who are willing to integrate carpooling in their service!

Learn more and dive into the developer docs over at [dev.blablacar.com](dev.blablacar.com)
