---
title: "Monthly Update July 2021"
description: ""
lead: ""
date: 2021-08-09T07:24:54+01:00
lastmod: 2021-08-09T07:24:54+01:00
draft: false
weight: 50
images: []
contributors: ["CloudUtil"]
---



Hello,

Another month, another community update. This one comes about a week late as I've been on a long vacation and some things had to give.

## Progress report - July 2021 ##

**AutoSpotting** - throughout my vacation I've been working a lot on a still unreleased branch that prepares AutoSpotting for being published on the AWS marketplace as a paid Docker product, charging a small percentage of the savings. This was way harder than I expected, as AutoSpotting is probably the only Lambda-based software being made available as paid Docker product so a lots of things had to be figured out and tweaked for making it work under the Marketplace constraints. There are many not-so-functional changes under the hood, such as calculating the generated savings, repackaging the Lambda to use Docker images in order to fit the Docker product requirements of the marketplace, charging a percentage of the savings amount through the Marketplace APIs from a new Fargate task(don't ask! ;-) ), etc. I've learned a lot while building this(I'll probably publish a technical blog post about it at some point) and it was a lot of fun and even though it basically works, I still have a number of things to figure out or update(especially docs). My goal is to have this published by the end of August, fingers crossed! I'll do my best to not cause much impact to the OSS users, but some things might be rough on the OSS side for a while after this code lands.

**EBS-Optimizer** - My main focus has been on AutoSpotting but for the first part of the month I actually focused on EBS Optimizer, where I spent a lot of time integrating it with the pricing API for metering the generated cost savings. This part turned out to be much easier on AutoSpotting, where we had all the pricing data already available, while on EBS Optimizer everything needed to be done from scratch using the pricing API, which is difficult at times. I also did some of the Marketplace integration work in parallel with AutoSpotting but it soon turned out intractable to do them in parallel so I then focused on AutoSpotting first. But some of the things I figured out on AutoSpotting will also apply here, and probably in a simplified way, I just haven't yet implemented everything.

**cloudutil.io** - no progress here this month, but I expect to work on it soon, in particular documenting these recent changes in detail.

## Plans for August and beyond ##

- My focus for August will be working towards the next stable release of AutoSpotting through the AWS Marketplace.
- If time allows I'll continue towards also publishing the first version of the EBS Optimizer on the AWS Marketplace, and then to open source it.
- I want to resume working on the cloudutil.io website, updating and adding all the existing AutoSpotting Markdown docs and creating new ones for the currently poorly/non-documented functionality.
- As I mentioned before, at work I'm also working on implementing certain features in the interest of my employer in the projects I build or contribute towards. Last month I added Spot information to [ec2instances.info](https://ec2instances.info), and after my vacation I'll work on AutoSpotting for implementing the capacity-optimized allocation strategy and proactive Spot instance replacements with Spot(with on-demand failover) in the event of instance rebalancing notifications.

I'll keep sharing my progress monthly as I work on these so stay tuned for more updates, I'm very excited about the future of my tools.

As always, I'm always eager to hearing from you, so feel free to reach out if you have any questions, feedback, or need any help. Just answer this email and I'll answer every single email I get from you.

I'd also love to see you in our next Community Zoom call, I'll share the invite soon.

Best regards,

Cristian
