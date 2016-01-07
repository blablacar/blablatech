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

Après investigation, il est ressorti le constat suivant: chaque load-balancer annoncait bien les même VIPs avec une med différente, et cette med était prise en compte par le switch ToR correspondant. 

Mais, la VIP étant réannoncée par le ToR à l'étage supérieur (aggregation), la med disparaissait (prévu dans eBGP quand on change d'AS: la [med est un attribut non-transitif](<http://www.cisco.com/c/en/us/support/docs/ip/border-gateway-protocol-bgp/13759-37.html>) ). 

Ce qui fait que les switches d'aggreg, et à fortiori les routeurs edge avaient plusieurs routes à disposition, avec un des as-path différents (donc pas éligibles au `bgp-multipath`). La conclusion était de choisir la route la plus ancienne, et c'est cohérent avec le comportement observé: un seul LB (le premier démarré en fait) recoit tout le traffic.

## Solutions envisagées

Plusieurs workaround ont été imaginés pour contourner ce problème:
* bgp multihop
* serveur de route
* utilisation des communautés
* ExaZK

### bgp multihop

La première idée que nous avons eue était de connecter les LB non pas aux ToR, mais aux switches d'aggreg. Il fallait pour cela utiliser le mode multihop d'eBGP: 

   router bgp 65002
     neighbor a.b.c.d remote-as 64xxx
       ebgp-multihop 2

Le problème rencontré avec cette solution: le next-hop pour les routes recues n'était pas connu dans l'IGP du routeur d'aggreg, la route était donc pas éligible à son insertion en RIB.

### serveur de route

L'idée suivante était de faire appel à un serveur de route. Au lieu de parler en BGP aux ToR, les LBs auraient alors parlé aux switches d'aggreg en mode route-server. Mais, nos switches d'aggregation sont des Cisco nexus 3064 et l'implémentation de BGP sur ce modèle n'inclue pas la directive `route-server-client`

### BGP community

Une autre piste: au lieu d'utiliser un attribut non transitif comme la med, nous avons envisagé d'utiliser les communautés BGP. Nous avons abandonné cette piste pour 2 raisons:
* complexité des route-map sur les switches d'aggreg pour prendre en compte ces communautés
* besoin de modifier le code source d'ExaBGP / healthcheck pour supporter les communautés à la place de la med.

### ExaZK

Quitte à modifier le code source d'healthcheck, nous avons trouvé plus rentable de faire notre propre plugin, et de le connecter directement à notre annuaire de service.

En effet, dans notre infra orientée containers, l'orchestration des différents containers (instanciation, determination du nb, placement, ...) s'appuie de manière importante sur ZooKeeper.

Il devenait évident pour nous que ExaBGP devait parler directement avec ZK pour savoir quoi annoncer aux routeurs BGP. Nous avons développé ce plugin: [ExaZK](<https://github.com/shtouff/exazk>)

<img src="/images/2016-01-06-bgp-routing-to-containers/new-infra2.png" class="block" style="width: 531px;" />

Le principe est le suivant:
* pour un service donné, disons `apex`, on fait tourner 3 pods 'load-balancer', sur 3 coreos différents, dans 3 racks différents (contraintes de placement d'un container)
* dans chacun de ces pods, le container `exabgp` fait tourner exazk
* chaque instance d'exazk annonce une seule VIP, pour laquelle cette instance fait authorite (param auth_ip)
* chaque instance écrit dans ZK le fait qu'elle est vivante, sous la forme d'un node ephemeral
* si une instance d'exazk meurt, son node ephemeral disparait, et les autres instances d'exazk s'en apercoivent.
* les instances restantes annoncent alors la VIP du node disparu, jusqu'à ce qu'il réapparaisse.

Un mode special, appelé maintenance, permet à un opérateur de mettre un service complet en maintenance: tous les LBs pour ce service arrêtent d'annoncer leurs VIPs. Le service est alors inaccessible.

Enfin, un local check, sur le même principe qu'`healthcheck` permet à chaque instance d'`exazk` de déterminer localement s'il est apte à annoncer sa VIP.

ExaZK est encore en cours de test, si le principe vous intéresse et que vous souhaitez le modifier pour vos besoins, n'hésitez pas à forker le projet sur [github](<https://github.com/shtouff/exazk>) !

Cheers
