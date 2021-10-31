---
title: "Monthly Update June 2021"
description: ""
lead: ""
date: 2021-07-04T11:54:03+01:00
lastmod: 2021-07-04T11:54:03+01:00
draft: false
weight: 50
images: []
contributors: ["CloudUtil"]
---

Hello,

Another month, another AutoSpotting community update.

## Progress report - June 2021

**EBS-Optimizer** - To be honest this month I haven't made as much progress on AutoSpotting as I had hoped, but I started a couple of new workstreams which kept me busy. The first is EBS-Optimizer, my new tool dedicated for EBS volume upgrades from GP2 to GP3, resulting in up to 20% better cost and often better performance than GP2 volumes.

I essentially just carved this functionality from AutoSpotting and made it into a generic tool that can be used by anyone using EC2 to reap these benefits. You can see more about it, including a live demo, in the latest recording of the AutoSpotting community call.
{{< video ratio="16x9" attributes="controls muted" mp4-src="https://autospotting.io/videos/AutoSpotting%20-%2024%20June%202021.mp4" >}}

A lot of it has been based from AutoSpotting ideas and code, so if you like AutoSpotting you'll probably also like EBS-Optimizer. It's still pretty basic but I have a few ideas on how to improve it, stay tuned for more.

EBS Optimizer is still closed source but I'm planning to open source it soon under the same OSL-3 license I use for AutoSpotting.

Let me know what you think about it, and especially if you're interested to try it out. Just answer this email and let's talk about it.

**cloudutil.io** - I've also started building a new [website](https://cloudutil.io) that will eventually host both AutoSpotting and EBS-Optimizer once it's ready. My main objectives with it are to make it look cleaner and more polished than autospotting.io, to have a home for more projects I may build going forward and a better host for the current AutoSpotting documentation, which is only accessible in a few markdown files scattered across the AutoSpotting Github repo, while the FAQ was available for a while now at autospotting.io.

There are still many things that need to be figured out and implemented, but so far I have an initial version of the front page, I migrated the AutoSpotting [FAQ](https://cloudutil.io/docs/autospotting/faq/) and a couple of other Markdown files so you can get a taste on how it will look and feel once ready. Web development isn't really my superpower so I would really appreciate any sort of feedback or help on it if you can spare a few cycles.

**AutoSpotting** - considering the other things I've been up to this month, this was a relatively slow month for AutoSpotting, but I've been progressing nicely towards the next stable release:

- I exposed the configuration of the EBS volume upgrade threshold all the way to the CloudFormation and Terraform infrastructure code.
- I made the event-based mode configurable globally, and also removed the per-ASG configuration for this functionality which was needlessly complicating the internal logic.
- I upgraded the library dependencies, including the instance type information to include also the latest X2gd Graviton-based EC2 instances.
- I've also re-tested AutoSpotting on both Graviton and Intel configurations and it worked pretty well with all these changes.
- I fixed a crash occurring for a certain configuration of the EBS root volumes, huge thanks to [@JanKulinski](https://twitter.com/JanKulinski) for reporting this issue and confirming my solution to it.

These changes have also been discussed in more detail in the latest recording of the AutoSpotting community call, which you've seen above but feel free to reach out if you have any questions.

## Plans for July

- My focus for July will be working towards the next stable release of AutoSpotting. I have a couple more things to tidy up and test scenarios to run, and then I want to make this release available through the AWS Marketplace in addition to Patreon. The main challenge is how to charge for Lambda tools through the Marketplace, since there's no official support for it, but I'll try to use Docker images to run the AutoSpotting Lambda when installed from the Marketplace. I'll probably also try this for the Terraform code, just for fun.
- I will also use these Marketplace learnings towards publishing the first version of the EBS Optimizer on the AWS Marketplace, and then to open source it. If time allows it I may also implement some few functionality, here are a few ideas I'd like to explore for the second release: backing up the initial/previous configuration in a tag, handling attach/detach actions of EBS volumes in order to to convert detached volumes to magnetic for additional savings and back to the previous configuration. Let me know if there's anything else you'd like to see in this tool.
- I want to keep working on the cloudutil.io website, adding all the existing AutoSpotting Markdown docs and creating new ones for the currently poorly/non-documented functionality.
- I'd like to also revamp the public roadmap of AutoSpotting, to create a new public roadmap for EBS-Optimizer, the cloudutil.io website and for collecting/prioritizing ideas for a few other tools I'd like to build afterwards. For now I'll probably just use Github Projects for this, with links to them from cloudutil.io. I'll let you know once this is ready and I'd love to get some prioritization input from you all.
- As I mentioned before, at work I'm also working on implementing certain features in the interest of my employer in the projects I build or contribute towards. The plan for this month is to add Spot information to [ec2instances.info](https://ec2instances.info), and once the AutoSpotting stable release is out I'll resume working on it for implementing the capacity-optimized allocation strategy and proactive Spot instance replacements with Spot(with on-demand failover) in the event of instance rebalancing notifications.

As I said, I'll share my progress monthly as I work on these so stay tuned for more updates, I'm very excited about the future of AutoSpotting and EBS-Optimizer and I have plenty of ideas for building new tools in this space.

As always, I'm always eager to hearing from you, so feel free to reach out if you need any help, have any suggestions or if you have any sort of feedback, just answer this email and I'll answer every single email I get from you.

I'd also love to see you in our next Community Zoom call in a couple of weeks, I'll share the invite soon.

Best regards,
Cristian
