---
title: "Continuous Deployment Trends"
date: "2013-10-14"
categories:
tags:
  - "ci"
  - "continuous_deployment"
---

This is a post putting together what companies are doing in the Continuous Deployment space, and what are the current trends. 

Deployments at Etsy
-------------------------

Here are some data about Etsy deploys to production (only data for half of the year 2013)

* Average Deploys per day: 35.75
* Average Authors per deploy: 7.01
* Average Commits per deploy: 11.25 

What we can read between lines is, if your "Average Deploys per day" is lower than 1 you're not doing Continuous Deployment, (you probably releasing often but not doing it continuously)

Some insides:

* Part of the Etsy Bootcamp is to deploy code in production, on your first day.
* Config System / Feature Switches: You have the ability to enable a feature, for group of users, a percentage of users, or for A/B testing

The typical dev cycle is:
Deploy your feature in production ASAP -> Enable for QAs and Admins -> Public Prototype (5%-10% of users) -> A/B Test (50% users) -> Full website

* Branching in Code: Use your configuration system, to keep your feature independent, and avoid to have long live branches.
* Versioning: There is no versioning and no rollback, you always push forward (You can disable features if something goes wrong)
* Experimentation: They have a platform to manage what features are used, and when they are in production. (First they use a tool "Launch Calendar", now more advanced unified launch management tool called "Catapult")
* Metrics: Detect if any of the changes goes wrong (statsd, ganglia, etc) 
* Start simple > Deploy ASAP > Experiment > Learn 

Etsy:

* [Etsy Deploys](http://www.infoq.com/presentations/etsy-deploy)
* [Etsy deployments and metrics](http://devslovebacon.com/conferences/bacon-2013/talks/bring-the-noise-continuously-deploying-under-a-hailstorm-of-metrics)

Deployments at IMVU
-------------------------

IMVU is "the inventor" of Continuous Delivery but also leading the "Lean Startup"

* Deploys per day: 50

Some insides:

* The time to deploy is around 20 minutes
* The feedback on features is always from real customers
* Metrics, as an essential part of the deployments
* Also branching in code and feature enabled/disabled

IMVU:

* [Continuous Deployment at IMVU](http://www.slideshare.net/bgdurrett/sds-2010-continuous-deployment-at-imvu)

Deployments at Quora
-------------------------
Quora is doing also well

* Deploys per day: 46

Some insides:

* From the developer's side, only a single command is required to push code to production: git push
* It takes six to seven minutes on average for a revision to start running in production.

Quora:

* [Continuous Deployment at Quora](http://engineering.quora.com/Continuous-Deployment-at-Quora)


How to enable Continuous Deployment
-------------------------
I highly recommend to watch this talk, because is a good summary of all the techniques:  

[Many Ways to Deploy Continuously](http://www.confreaks.com/videos/2365-mwrc2013-the-many-ways-to-deploy-continuously)

Some common patterns for devs are:

* Branching in Code
* Versioning the Database Schema
* Deploy to handle multiple versions at the same time (a deploy on 1000 machines could take a while, be ready to have multiple versions at the same time)
* Ship an image (AMIs, dokus, docker containers ...) isolate problems and makes your environment predictable and testable

Other links
-------------------------

* [Continuous Testing at Google](http://www.infoq.com/presentations/Continuous-Testing-Build-Cloud)
* [Martin Fowler on Continuous Delivery](http://martinfowler.com/bliki/ContinuousDelivery.html)
* [Continous Delivery Patterns](http://www.infoq.com/articles/Continous-Delivery-Patterns)
