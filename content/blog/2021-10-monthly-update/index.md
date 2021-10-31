---
title: "Monthly Update October 2021"
description: ""
lead: ""
date: 2021-10-31T20:36:12+01:00
lastmod: 2021-10-31T20:36:12+01:00
draft: false
weight: 50
images: []
contributors: ["CloudUtil"]
---

Hello,

As usual, here's my latest monthly report.

Going forward I'll release these reports here on the CloudUtil
[blog](https://cloudutil.io/blog/) and I'll only share the link through my
mailing list and our new
[Slack](https://join.slack.com/t/cloudutil/shared_invite/zt-xodcoi9j-1IcxNozXx1OW0gh_N08sjg)
#announcements channel.

If you want to be notified when I write a new one, don't forget to sign up to
the CloudUtil newsletter shown at the bottom of the page, or join us on Slack.

I also published as blog posts most emails I had sent to the AutoSpotting
newsletter over the last year, so you can see how it changed over this time. I
had to edit them a bit to fix URLs and clean them up a bit, bear with me in case
there were any issues.

## AutoSpotting

- This month I further polished and merged a massive change to the way
  AutoSpotting works under the hood, in particular the way we launch the Spot
  instances. This was explained in a lot of detail in the previous monthly
  [update](https://cloudutil.io/blog/monthly-update-september-2021/) and I won't
  repeat it all here, I'll just state its main benefit, namely that in the new
  default configuration the Spot instances launched by AutoSpotting will be
  likely to run for longer time before being interrupted.
- In addition to this massive change there were just a handful of small
  buildsystem changes and a minor bugfix.
- The Terraform code was updated to expose the ability to disable the
  Rebalancing Recommendation events.

These changes aren't yet available in a stable release, but I'll try to release
a new stable build sometime in November that includes them, stay tuned.

## EBS Optimizer

No changes over the last month.

## Websites

- I performed a few changes on [autospotting.io](https://autospotting.io),
  mainly further cleaning up the copy and adding new testimonials. I also
  disabled the annoying popup that asked for installation of AutoSpotting from
  the AWS Marketplace when scrolling to the bottom of the page.
- [cloudutil.io](https://cloudutil.io) has also seen a few copy changes and
  cleanups, as well as the addition of the blog section, which was populated
  with previous emails sent to the newsletter. Speaking of newsletters, I also
  integrated the same Sumo newsletter I use on
  [AutoSpotting.io](https://AutoSpotting.io).

## Income report

I've finally received the first income from the AWS Marketplace, covering
September. After deductions and fees I received 37.98 EUR in my bank account. In
addition to this I received 370 USD from Patreon, which unfortunately was
affected by some payment issues in India, where payments were impacted by some
new banking regulations. If you're based in India and using Patreon, I urge you
to switch to the [AWS
Marketplace](https://aws.amazon.com/marketplace/pp/prodview-6uj4pruhgmun6) to
avoid such issues.

Speaking of the Marketplace, the number of marketplace subscribers kept growing
and reached 5 active subscribers, and a few more inactives. All in all I expect
the October Marketplace income to be around 100-120EUR. I also got inquiries
from relatively large companies which I hope to materialize in November.

I'd like to extend a warm "Thank you!" to all of you who are purchasing
AutoSpotting supported builds, your continuous support motivates me to keep
improving it further.

## Plans for November and beyond

- Stable Marketplace release that includes the recent AutoSpotting features.
- Add more docs on [cloudutil.io](https://cloudutil.io) about both EBS Optimizer
  and AutoSpotting
- Proactive Spot-to-Spot instance replacements in the event of instance
  rebalancing notifications, with ICE-proof on-demand failover across multiple
  instance types

I'm incredibly excited about the future of my tools and I'll keep sharing my
progress monthly as I work on them, so stay tuned for future updates.

I'm always eager to hearing from you, so feel free to reach out if you have any
questions, feedback, or need any help. Just ping me on Slack and I'll personally
answer every single message I get from you.

Best regards,

Cristian
