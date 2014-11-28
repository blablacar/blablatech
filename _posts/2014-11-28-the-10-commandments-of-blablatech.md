---
layout:     post
title:      "The 10 commandments of BlaBlaTech"
tags:       culture
authors:    [nicolas-schwartz, christian-jennewein]
---

Welcome to the new BlaBlaTech blog (build with fantastic [Jekyll](http://jekyllrb.com/), for comments please use our [Twitter](https://twitter.com/BlaBlaCarTech))! This first article will actually consist of an updated version of our 10 commandments, a **top 10 list of best practices combined with valuable advices** that guide our tech team for many years now.

Those commandments helped to shape and continue to help maintaining our healthy development culture, a first-class citizen in our team. And although they are our commandments, chances are that **some of those might also help you**.

Exceptionally we also publish [a french version, further down](#french-version).

### 1. Thou shalt always think "Built To Last"
 * Do not make a dirty fix when you know there is a more robust solution to the problem
 * Comment your code so that it's understandable to other members of the team (and yourself) when they come across it later
 * Learn from your mistakes and communicate on them ("Fail, Learn, Succeed")

### 2. Thou shalt never make changes directly in prod (Code of DataBase)
 * Respect the rollout process:<br />[Dev + test] => [Preprod + tests] => [Prod + tests]
 * Backoffice and APIs are not exempt from this rule

### 3. Thou shalt respect the following principles of development: KISS (Keep it simple, stupid) and SoC (Separation of Concerns)
 * No wording to be displayed outside the translation table
 * No style information outside the CSS files
 * No HTML in your PHP files
 * No Javascript validation without a corresponding server-side process

### 4. Thou shalt not take the team for your beta testers
 * Do not submit something to other team members without having properly tested it on multiple browsers and multiple test cases ("Never assume, always check")
 * Use your common sense and pass a critical eye over your code quality, architecture, design and ergonomics before delivering

### 5. Thou shalt put yourself in the place of the user
 * Rejoice when a user writes to report a problem ("The member is the boss")
 * Keep in mind that for every user who complains, 100 have had the same problem and not told us anything. A user who complains is, from his point of view, always right.
 * Put yourself in the user's shoes by ridesharing ("Think it, build it, use it")
 * Do not roll out something in production that goes against your common sense

### 6. Thou shalt respect our working methodology
 * Do not deliver a feature for validation without first checking that it includes all the features originally requested by the PO
 * Do not let the PO say the same thing twice
 * Do everything possible to fulfill your team's commitments on the current sprint
 * Do not start a big development without first discussing it with the team, PO and Scrum Master included ("Done is better than perfect")
 * Do not bear any grudges - discuss any issues during the retrospective

### 7. Thou shalt ensure the prod functions properly
 * Issues in production take precedence over everything else
 * Always tell your PO which prod bug you're working on

### 8. Thou shalt foster a positive atmosphere within the team
 * Reply to any messages you receive that contain a question
 * Set comments in your commits
 * Document and share information about new innovations that you come up with ("We are passionate, we innovate")
 * Share your knowledge beyond the boundaries of the team if the opportunity arises ("Share more, learn more")
 * Always use collaborative tools
 * Give quick and constructive answers to the PR team's questions

### 9. Thou shalt understand how your work complements other teams
 * Techs are attentive and talented people, otherwise they would not be here
 * People from Marketing, Member Relations and Communication are the same...
 * Always help people when they come to see you, or if you can't help, direct them to the person who can!

### 10. Thou shalt enjoy your work, otherwise it's not worth bothering with!
 * If you're no longer enjoying your work, talk to your manager and/or Francis
 * This approach is designed to help you enjoy everything you do, all the time!

## <a name="french-version"></a>Version Française

### 1. Tu penseras toujours "Built To Last"
 * Tu ne feras pas de dirty fix lorsque tu connaîtras une solution robuste au problème
 * Tu commenteras ton code en pensant aux autres membres de l’équipe (et pour toi-même) qui le découvriront
 * Tu apprendras de tes erreurs et communiqueras dessus (« Fail, learn, succeed »)

### 2. Tu ne feras jamais de modif en prod (Code ou Base)
 * Tu respecteras le processus de mise en production :<br />[ dev + tests ] => [ preprod + tests ] => [ prod + tests ]
 * Le backoffice et les API ne se soustraient pas à cette règle

### 3. Tu respecteras les principes de développement KISS (Keep it simple, stupid) et SoC (Separation of Concerns)
 * Pas de wording destiné à l'affichage en dehors de la table de traduction
 * Pas d'information de style en dehors des fichiers CSS
 * Pas d’HTML dans tes fichiers PHP
 * Pas de validation Javascript sans un traitement côté serveur correspondant

### 4. Tu ne prendras pas l’équipe pour tes beta-testeurs
 * Tu ne soumettras pas aux autres membres de l'équipe une livraison sans l'avoir honnêtement testée sur plusieurs navigateurs et dans plusieurs cas de test (« Never assume, always check »)
 * Tu garderas ton œil critique et ton bon sens en qualité de code, architecture, design et en ergonomie avant de livrer

### 5. Tu sauras te mettre à la place de l'utilisateur
 * Tu te réjouiras lorsqu’un utilisateur t’écrira pour nous signaler un problème (« The member is the boss »)
 * Tu garderas en tête que pour un utilisateur qui se plaint, 100 ont eu le même problème et ne nous ont rien dit : un utilisateur qui se plaint a toujours raison, de son point de vue,
 * Tu feras du covoiturage pour te placer en situation « utilisateur » (« Think it, build it, use it »)
 * Tu ne livreras pas en prod quelque chose qui ira à l’encontre de ton bon sens

### 6. Tu respecteras notre méthodologie de travail
 * Tu ne livreras pas une fonctionnalité pour validation sans avoir vérifié au préalable qu'elle intègre toutes les caractéristiques transmises initialement par le PO
 * Tu ne feras pas répéter le PO
 * Tu feras le maximum pour tenir les engagements de ton équipe sur le sprint en cours
 * Tu n’entameras pas de développement conséquent sans en discuter avant avec l’équipe (PO / Scrum Master compris) (« Done is better than perfect »)
 * Tu ne garderas pas de rancoeur pour toi et tu sauras l’expliquer pendant la rétrospective

### 7. Tu veilleras au bon fonctionnement de la prod
 * Les problèmes de prod sont prioritaires sur tout le reste
 * Ton PO, informer tu devras, pour lui indiquer que sur un bug de prod tu travailles

### 8. Tu favoriseras l’émulation positive dans l’équipe
 * Tu répondras à tout message qui t'es destiné et qui contient un point d'interrogation
 * Tu placeras toujours des commentaires dans tes commits
 * Tu écriras des documentations et partageras l’information sur les innovations que tu apporteras (« We are passionate, we innovate »)
 * Tu partageras tes connaissances au delà des frontières de la Team si l’occasion se présente (« Share more, learn more »)
 * Tu utiliseras les outils collaboratifs mis en place
 * Tu répondras rapidement à une PR et toujours de manière constructive

### 9. Tu concevras ton activité comme complémentaire de celle des autres équipes
 * Les techs sont des gens bien et sensés, sinon ils ne seraient pas là
 * Les gens du marketing, des Relations Membres et de la comm’ aussi
 * Ouvertement tu les accueilleras si tu peux les aider et tu les aiguilleras vers la bonne personne sinon

### 10. Tu te feras plaisir, sinon ce n’est pas la peine de continuer !
 * Si ton activité ne te plait pas ou plus, tu en discuteras facilement avec ton manager et/ou Francis
 * Sur cette base, ton activité te plaira donc tout le temps !
