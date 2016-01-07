---
layout:         post
title:          BGP routing to containers in BlaBlaCar
tags:           [network, BGP, containers, architecture]
authors:        [remi-paulmier]
description:    Impact de l'organisation en containers sur le routage du traffic public avec BGP 
---

# Contexte

Jusqu'à présent, le routage du traffic public jusqu'aux serveurs frontaux était assuré via des serveurs particuliers, appelés load-balancers dans notre infrastructure. Ces serveurs sont constitués autour de `nginx` pour l'off-loading SSL, `varnish` pour le reverse-proxying, et `ExaBGP` pour la communication avec nos routeurs BGP.

L'arrivée des `containers` avec `rkt` dans notre infrastructure nous a poussé à revoir dans le détail l'organisation de ces composants. C'est l'objet de cet article.

# Infra existante

Pour l'architecture existante, je m'étais inspiré de l'excellent article de Vincent Bernat: [High availability with ExaBGP](<http://vincent.bernat.im/en/blog/2013-exabgp-highavailability.html>)

Appliqué à notre infra, ca donnait cela:
<img src="/images/2016-01-06-bgp-routing-to-containers/old-infra1.png" class="block" style="width: 337px;" />

Chacun des load-balancers fait tourner [`ExaBGP`](<https://github.com/Exa-Networks/exabgp>), qui communique avec les routeurs BGP. Exemple avec l'un d'entre eux:

{% highlight python %}
group neighbors {
  neighbor 91.238.131.1 {
    router-id 91.238.131.5;
    local-address 91.238.131.5;
    local-as 65001;
    peer-as 202069;
  }
  neighbor 91.238.131.2 {
    router-id 91.238.131.5;
    local-address 91.238.131.5;
    local-as 65001;
    peer-as 202069;
  }
}
{% endhighlight%}


On utilise le plugin `healthcheck`, qui va, sur chaque LB, réaliser un test de dispo de la resource, puis si le test est concluant, annoncer des VIPs:

{% highlight python %}
process healthcheck-apex {
  run /etc/exabgp/processes/healthcheck.py --config /etc/exabgp/healthcheck-apex.conf;
}
{% endhighlight%}

Regardons le détail de healthcheck-apex.conf:

{% highlight python %}
name = apex
interval = 10
fast-interval = 1
command = curl -sf http://127.0.0.1/healthcheck
ip = 91.238.131.166
ip = 91.238.131.167
{% endhighlight %}

Sur le second LB, la config est la même, à l'exception de l'ordre des VIPs:

{% highlight python %}
name = apex
interval = 10
fast-interval = 1
command = curl -sf http://127.0.0.1/healthcheck
ip = 91.238.131.167
ip = 91.238.131.166
{% endhighlight %}

Pourquoi donc ?

C'est lié au fonctionnement d'`healthcheck` Pour chaque VIP spécifiée, healthcheck va faire faire une annonce à ExaBGP, en incrémentant la med d'une valeur arbitraire (1 par défaut). Ex sur le premier LB:

    announce route 91.238.131.166/32 next-hop self med 100
    announce route 91.238.131.166/32 next-hop self med 101

Ce qui fait que chaque LB annonce les même VIPs, mais avec un *poids* (une med) différent. Du coup, les routeurs ne routent le traffic pour une VIP donnée que vers un seul LB. 

Il suffit ensuite de publier ces 2 VIPs dans le DNS pour un service donné (ici *apex*), pour obtenir un service hautement disponible.

En cas de panne d'un LB, celui-ci n'annonce plus les VIPs, mais les routeurs connaissent une autre route pour ces VIPs vers le second LB, donc le service continue de fonctionner.

# Application aux containers

Dans notre nouveau datacenter, nous utilisons exclusivement les containers rkt comme unité d'exécution de base. Tous les serveurs sont identiques, et ne font fonctionner qu'un seul OS: CoreOS.  Cette plate-forme est donc industrialisée au maximum. 

D'un point de vue réseau, nous en avons profité pour sortir du modèle commuté, puisque nous n'avons plus besoin de VLAN. L'isolation se fait par des ACLs générées automatiquement et déployées sur les CoreOS (sur le même principe que les security groups d'AWS).

Nous avons donc mis en place BGP de bout en bout, afin de router le traffic jusque dans les switches top-of-rack. Un dessin vaut mieux qu'un long discours:

<img src="/images/2016-01-06-bgp-routing-to-containers/new-infra1.png" class="block" style="width: 418px;" />

Puis dans chaque rack, nous avons un CoreOS qui est éligible à porter un container de load-balancing. Ce container est responsable de faire tourner ExaBGP, et d'annoncer les VIPs qui vont être prises en charge par ce LB. 

On retrouve alors la config Exabgp suivante. Ex dans le rack 1:

{% highlight python %}
group neighbors {
  neighbor 10.20.151.129 {
    router-id 10.20.151.147;
    local-address 10.20.151.147;
    local-as 64147;
    peer-as 64001;
  }
  neighbor 10.20.151.130 {
    router-id 10.20.151.147;
    local-address 10.20.151.147;
    local-as 64147;
    peer-as 64001;
  }
}
{% endhighlight%}

## Problème rencontré

Lors des premiers tests, nous nous sommes assez vite rendus compte d'un problème: pour un service donné, qq soit le nb de LB démarrés, un seul recevait tout le traffic.

## Solution: exazk



