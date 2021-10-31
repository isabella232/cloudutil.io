---
title: "Monthly Update April 2021"
description: ""
lead: ""
date: 2021-05-01T21:48:07+01:00
lastmod: 2021-05-01T21:48:07+01:00
draft: false
weight: 50
images: [""]
contributors: ["CloudUtil"]
---

Hello,

It's been a while since the previous status update and also I really missed seeing some of you like I did in our monthly webinars.

I decided to give a status update for the last few months and considering it's the beginning of the month going forward I'll do my best to do this on a monthly basis, sharing the progress since the previous month and what's coming soon.

A more comprehensive status update including news related to the AutoSpotting Stable builds will be released at the same time to my private [Patreon](https://www.patreon.com/cristim) supporters, so if you're also a supporter you can probably just read that one instead.

I'm also trying to resume the AskMeAnything-style webinars around the middle of the month, next week I'll set up an LinkedIn event for the next session, stay tuned for the invites.

## Progress report

Ever since December, due to my current circumstances(intermittent pandemic lockdowns and team change at work which kept me busy with onboarding work) I've been struggling to find time to meaningfully work on AutoSpotting.

Having said this limited involvement, there are still a few exciting changes that landed over the last few months:

- migrated the CI to Github Actions, which is much more stable than the last few months of using Travis and helped improve the pull request quality checks
- fixed a few critical bugs on the logic that suspends AutoScaling processes when doing event-based instance replacements, which previously weren't cleaned up properly
- integrated CodeQL for enhancing the pull request quality checks
- helped land a major contribution by Alberto (mello7tre on Github), which simplifies the event handling logic, introducing a FIFO SQS queue for serializing ASG operations (such as temporarily extending the maximum capacity and suspending AutoScaling processes) instead of calling an auxiliary Lambda function with concurrency 1. The code and internal architecture is significantly  cleaner and hopefully more robust, but this is very new and I'll need some more testing.
- A few Terraform code bug fixes, but Terraform is probably broken because of this recent SQS change(Docker/K8s is also probably broken as well but for longer time)

## Other updates

Also when it comes to my time to work on AutoSpotting, I'm happy to announce that starting from this week I'm working 1-2 days a week on AutoSpotting as part of my daily job, mainly for new feature development in the interest of my current employer.

It also looks like I have some contributors chiming in occasionally, such as Alberto was with this recent SQS FIFO change, and also others expressing interest to help out going forward.

## Future work

- My focus for the next few weeks will be working towards the next stable release for my Patreon supporters, consisting mainly on testing and bug-fixing work.
- I'll also work on finally landing the GP2->GP3 automated storage volume upgrade logic(currently available in an experimental branch, but not fully configurable), which I'm very excited about.
- As I mentioned, at work I'll focus on implementing certain features in the interest of my employer. The plan so far is to work on adding support for multiple Spot allocation strategies (including the AWS-recommended capacity-optimized), and also to enhance the Spot termination handling logic to immediately launch Spot instances ideally without having to run any On-demand provisioned by the ASG using the Launch Configuration. If there's no Spot capacity available AutoSpotting would also try to launch On-Demand instances across other multiple instance types, in order to avoid causing ICEs on the original on-demand instance type.

These should keep me busy over the next few months, but as I said, I'll share my progress monthly as I work on these so stay tuned for more updates, I'm incredibly excited  about the future of AutoSpotting.

As always, I'm always eager to hearing from you, so feel free to reach out if you need any help, have any suggestions or if you have any sort of feedback.

Best regards,
Cristian
