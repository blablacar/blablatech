---
layout:         post
title:          How did Google perceive our SEO migration to blablacar.fr?
tags:           organisation
authors:        [jerome-moussay, tony-fouchard]
description:    The main goal was to unify our brand domain names everywhere, the SEO challenge was to make sure traffic loss would be kept to a minimum.
---

A website migration is always a risky task. Just as a reminder, we changed our brand name in France from covoiturage to 
BlaBlaCar in April 2013. We decided to switch the domain name only beginning June 2015 because we 
thought it was time do it in France: *blablacar* is now much more popular than the term *covoiturage*. The main goal 
was to unify our brand domain names everywhere, the SEO challenge was to make sure traffic loss would be kept to a minimum.

## What we did?

- Set up 301 redirects for all pages from covoiturage.fr to blablacar.fr (e.g. [http://www.covoiturage
.fr/trajets/rennes/](http://www.covoiturage.fr/trajets/rennes/) to [https://www.blablacar.fr/trajets/rennes/](https://www.blablacar.fr/trajets/rennes/)), and update all internal links.
- Warn Google that the domain has moved in Google Webmaster Tools.
- Update external links on most important websites. The 1st thing was to update the links on social platforms & on 
partner websites. Of course, you can’t update everything and this can be a very long (and useless in certain cases) 
task to contact all websites.
- Monitor the changes in terms of crawling, indexation, rankings, traffic and links.

## How did we proceed to migrate from a technical point of view?

### Massive redirects at scale

We started by checking the capacity of our redirector farm to treat a lot of HTTP(S) redirects and we ensured we were able to do reverse proxy caching of 301 responses for future covoiturage.fr responses.

Scaling up our infrastructure is easy because we can boostrap new machines with a chef role and machines become quickly able to serve extra requests, that is the power of infrastructure automation!

### Enabling www.blablacar.fr

We started by adding www.blablacar.fr to our plateform SSL certificates and deploy them accross all our delivery nodes.

After that, we added blablacar.fr in the HSTS browsers preload lists to register this domain as a full HTTPS one: at 
a member level, it makes the website faster avoiding to have server redirects extra round-trips, i.e. browser calls 
HTTPS URLs instead of HTTP ones.

The next step consisted in serving the website using an HTTP Host header with value *www.blablacar.fr*.

From our load balancers/reverse proxy cache levels, we had before migration:
 
 - www.covoiturage.fr routed to our application backends farm
 - www.blablacar.fr routed to our redirector backends farm

So, the steps to serve www.blablacar.fr were:

 - keeping serving www.covoiturage.fr with application backend farm
 - routing www.blablacar.fr to application backend farm at a reverse proxy level (no incoming traffic for the moment)
 - decreasing in advance www.blablacar.fr and www.covoiturage.fr DNS TTL, then updating www.blablacar.fr to point to 
 our application infrastructure IPs
 - purging the reverse proxy on *www.blablacar.fr*
 - routing www.covoiturage.fr to our redirectors
 - purging www.covoiturage.fr

This is how it behaved during migration:

    1. www.covoiturage.fr => 200,    www.blablacar.fr => 301 to www.covoiturage.fr
    2. www.covoiturage.fr => 200,    www.blablacar.fr => 200 (during few minutes)
    3. www.covoiturage.fr => 301 to  www.blablacar.fr => 200

## How did Google react in terms of crawling?

Right after the migration, Google crawled massively the pages on the domain blablacar.fr and he slowed down once they have all been discovered. We’ve now recovered the same crawl volumes as we used to have before the migration.

<p class="text-center">
    <img src="../../images/2015-07-23_how-did-google-perceive-our-seo-migration-to-blablacar-fr/2015-07-23-crawl-volume-migration-fr.png" alt="Crawl volume migration" />
    <span class="img-caption">Source: Botify</span>
</p>

As predicted, Google’s crawl on covoiturage.fr is decreasing with time but will certainly never stop because there will always be some remaining websites linking to covoiturage.fr. That’s why redirects should always stay active on covoiturage.fr.

## How did Google react in terms of indexation?

- Despite the redirects, some URLs of covoiturage.fr and blablacar.fr were indexed & positioned in Google at the same time.

<p class="text-center">
    <img src="../../images/2015-07-23_how-did-google-perceive-our-seo-migration-to-blablacar-fr/2015-07-23-indexation-migration-fr.png" alt="Indexation migration" />
</p>

- During a short period, we also had some URLs of our old platform that disappeared more than 1 year ago back in the search results! And also some URLs of our mobile website appearing in Google’s Desktop results.

<p class="text-center">
    <img src="../../images/2015-07-23_how-did-google-perceive-our-seo-migration-to-blablacar-fr/2015-07-23-indexation-issues-migration-fr.png" alt="Indexation issues migration" />
</p>

- It took 1 week to get our sitelinks back on the brand name.

<p class="text-center">
    <img src="../../images/2015-07-23_how-did-google-perceive-our-seo-migration-to-blablacar-fr/2015-07-23-sitelinks-blablacar-fr.png" alt="Sitelinks BlaBlaCar" />
</p>

- We now have all new URLs indexed in Google. There are still a lot of covoiturage.fr URLs in Google’s index because it requires time to clean them.  

<p class="text-center">
    <img src="../../images/2015-07-23_how-did-google-perceive-our-seo-migration-to-blablacar-fr/2015-07-23-volume-indexation-google-fr.png" alt="Volume indexation Google" />
</p>

## Impact in terms of ranking?

- On our top axis pages, it took about 3 weeks to get everything back to normal on keywords “ridesharing + city” 
(e.g. covoiturage paris) & “ridesharing + axis” (e.g. covoiturage paris nantes).

<p class="text-center">
    <img src="../../images/2015-07-23_how-did-google-perceive-our-seo-migration-to-blablacar-fr/2015-07-23-rankings-covoiturage-fr.png" alt="Rankings covoiturage" />
    <span class="img-caption">Source: Myposeo</span>
</p>

- Same thing for axis only (paris nantes), we even get better results after the migration:

<p class="text-center">
    <img src="../../images/2015-07-23_how-did-google-perceive-our-seo-migration-to-blablacar-fr/2015-07-23-rankings-axis-fr.png" alt="Rankings axis" />
    <span class="img-caption">Source: Myposeo</span>
</p>

## Impact in terms of traffic?

- SEO traffic on the whole website didn't decrease during the migration, which is definitely the most satisfying 
result we could encounter.

<p class="text-center">
    <img src="../../images/2015-07-23_how-did-google-perceive-our-seo-migration-to-blablacar-fr/2015-07-23-trafic-migration-fr.png" alt="Traffic migration" />
    <span class="img-caption">Source: Botify</span>
</p>

- SEO traffic on city & axis pages actually decreased the 2 weeks after the migration because of the loss of rankings
 on some queries, but traffic on those pages get back to its normal growth tendency.

<p class="text-center">
    <img src="../../images/2015-07-23_how-did-google-perceive-our-seo-migration-to-blablacar-fr/2015-07-23-trafic-axis-migration-fr.png" alt="Traffic axis migration" />
</p>

## Conclusion

We were pretty confident about that migration but we actually didn't expect so many (strange) moves in Google’s 
results. Of course you can always make some assumptions why it behaves this way, but you actually have no clue of 
what is exactly going to happen.

The lesson is that even if you do the maximum of required optimizations, you always have to expect some disturbances 
in your SEO when you do a website migration. Monitoring the changes with data is the best way to understand what’s 
really going on and react if things don’t go the right way.