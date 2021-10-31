---
title: "Monthly Update August 2021"
description: ""
lead: ""
date: 2021-08-29T10:13:42+01:00
lastmod: 2021-08-29T10:13:42+01:00
draft: false
weight: 50
images: []
contributors: ["CloudUtil"]
---



Hello,

As usual, here's my monthly progress report, this time covering August 2021.

## AutoSpotting ##

As I mentioned in my previous report, I've been working on releasing the new AutoSpotting stable build on the AWS Marketplace. After many months of hard work, I'm happy to announce that the release finally landed on the Marketplace a few days ago, and it's probably the first serverless paid product built with Lambda available on the Marketplace.

This is the first stable release since November 2019, so the amount of changes and bug fixes is staggering, and under the hood the code has been all but rewritten since the previous stable release. To list just a few major functional changes included in this stable release:

- Event-based instance replacement, having new OnDemand instances replaced with Spot immediately after launch, reducing the churn observed by your AutoScaling group and speeding up the replacements of multiple instances launched at once as much as possible.
- Automatically upgrade EBS volumes from GP2 to GP3 and IO1 volumes to IO2 for increased performance and/or lower cost, such as 20% less for the GP2 EBS volumes below 170GB.
- Support for new instance types, all instances available as of July 2021 are now supported, including Graviton2, if you use it in the Launch Configuration/template.
- Handling the EC2 Rebalancing Recommendation events, for earlier/more proactive instance replacement.

Notes:

- For now it can only be installed from CloudFormation, but I'll also add support for Terraform soon.
- The pricing changed from the previous $29/month flat fee to a small percentage of 5% of the generated savings. You're charged hourly in increments of $0.001/h for each $0.02/h saved, free of charge below $0.02/h and for intermediary values over the largest multiple of $0.02/h. The break-even point compared to the previous stable version is around $600/month saved(actually it will be around $1000, as you'll see below), so if you're saving less than that, this release will cost you less than before. It's now also completely pay-as-you-go and charged on your AWS bill without the need to sign up on Patreon, which added friction to many potential users and caused problems when people switched jobs or their credit cards used for Patreon expired. It's also more inexpensive or even free for smaller users or if just trying it out, and  for the larger users still much more affordable than any of the alternative solutions in this space and the only one also fully available as open source.
- I will stop offering ZIP files with nightly binaries, going forward the only binaries I'll offer will be the ones available on the AWS marketplace
- I plan to release new stable builds on a regular basis(probably quarterly).
- Users of the former stable release can use it indefinitely, but I strongly recommend to upgrade, as there are so many enhancements and bug fixes.
- As a token of appreciation for my current Patreon supporters/stable build users, I'm working on a special build with a reduced rate of 3% of the hourly savings, so stay tuned to your Patreon feed.

You can check it out [here](https://aws.amazon.com/marketplace/pp/prodview-6uj4pruhgmun6) on the AWS Marketplace. Please let me know if you have any further questions or feedback about this stable release.

In addition to the work on this stable release, I've also recently started working on a large refactoring that's switching the way of launching Spot instances to use instant EC2 Fleets instead of RunInstances API calls. The benefit of this change is the fact that we'll be able to select from the multiple available allocation strategies, such as the capacity-optimized, which increases considerably the uptime of the launched Spot instances. This feature isn't yet ready and wasn't included in this stable release, but it's been progressing nicely so far and already available on a Github branch if you want to try it out. I have it already running and currently working on testing it further, extending the automated test coverage, and then the plan is to make it configurable.

## EBS Optimizer ##

It was also recently released on the AWS Marketplace, at about the same time with AutoSpotting. In case you missed my previous emails about it, EBS Optimizer builds upon AutoSpotting's EBS volume upgrade functionality but if offers a more comprehensive implementation that I plan to extend further over time.

It's also designed to be a more general-purpose tool, meaning that you should be able to use it for any sort of EC2 workloads, even if you're not using AutoSpotting at all. It looks and feels much like AutoSpotting, so if you liked AutoSpotting, you'll probably also like it. If you run any EC2 instances, chances are that EBS Optimizer will save you some money, which can add up to significant amounts at scale.

EBS Optimizer is also charged at $0.001/h for each $0.02/h saved, so you'll only pay a small fraction of the generated savings, also not more than 5% and it's also usually free of charge or very inexpensive for the small users and much more inexpensive than any of the alternatives in this space.

You can check it out [here](https://aws.amazon.com/marketplace/pp/prodview-ryzl67mmq3ghk) on the AWS Marketplace.

[**cloudutil.io**](https://cloudutil.io) - no progress this month, but I expect to work on it soon.

## Plans for September and beyond ##

- Open Source EBS Optimizer
- Update Terraform code for AutoSpotting to support the new version available through the AWS Marketplace
- Special AutoSpotting stable build for Patreon supporters that offers a reduced rate of 3% of the generated savings
- Finish the replacement of RunInstances with instant EC2 Fleets for launching Spot instances with the different allocation strategies, such as capacity-optimized.
- Add more docs on cloudutil.io about both EBS Optimizer and AutoSpotting
- Proactive Spot-toSpot instance replacements in the event of instance rebalancing notifications, with ICE-proof on-demand failover across multiple instance types

I'm incredibly excited about the future of my tools and I'll keep sharing my progress monthly as I work on these, stay tuned for future updates.

As always, I'm always eager to hearing from you, so feel free to reach out if you have any questions, feedback, or need any help. Just answer this email and I'll personally answer every single email I get from you.

Best regards,

Cristian
