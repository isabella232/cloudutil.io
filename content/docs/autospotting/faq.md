---
title: " Frequently Asked Questions"
description: "Answers to frequently asked questions about AutoSpotting."
lead: "Answers to frequently asked questions about AutoSpotting."
date: 2020-10-06T08:49:31+00:00
lastmod: 2020-10-06T08:49:31+00:00
draft: false
images: []
menu:
  docs:
    parent: "autospotting"
weight: 600
toc: true
---

<!-- markdownlint-disable MD025 -->

In case you haven't found the answer for your question here please feel free to
contact us on [gitter](https://gitter.im/cristim/autoSpotting), our vibrant
community will most likely help you.

<!-- markdownlint-disable MD026 -->

<!--
Feel free to add here any questions related to the AutoSpotting project and to
edit existing items if you notice any inaccuracies. Once the list is in a decent
shape it will be added to the official project documentation under a new FAQ
section.

For editing please try to use Markdown syntax or simply just write unformatted
text, we can take care of the formatting later.
-->

## What is AutoSpotting?

AutoSpotting is a tool implementing an automation of the Amazon AWS EC2 Spot
pricing model, which gives you much cheaper EC2 Spot instances.

## What are Spot instances, what is Spot and how does it work?

Cloud providers such as Amazon AWS need to always have some spare capacity
available, so that any customer willing to launch new compute machines would be
able to do it without getting errors. Normally this capacity would be sitting
idle but still consuming power until someone needs to use it.

Instead of wasting this idle capacity, Amazon initially created a marketplace
for anyone willing to pay some money in order to use these machines, but
knowingly taking the risk that this spare capacity may be taken back within
minutes when it needs to be allocated to an ordinary on-demand user.

This is now known as *Spot*, and these interruptible compute machines are
called *Spot instances*.

From the functionality perspective Spot instances are identical to on-demand
instances, the only difference is the fact that they can be interrupted.

## How are Spot prices calculated and how much are they?

The Spot  market automatically computes a price based on the long-term supply and
demand trends, updated daily with a small increment, and everyone using the same
AvailabilityZone within a region pays the same price for a given
instance type. The prices are 60-90% less than the usual on-demand hourly prices.

## How stable are Spot instances?

Amazon publishes interruption rates for each instance type on their [Spot
Instance Advisor(https://aws.amazon.com/ec2/spot/instance-advisor/) tool, and
states that only about 5% of the instances are interrupted within the last month.

In many cases the Spot capacity is so stable that Spot instances may run for
weeks or even months at very steep savings, so it's basically a waste of money
not to run Spot instances for workloads that are tolerant to occasional
interruptions.

Historically the Spot marker used to be much more volatile, but it changed a lot
over the last few years so nowadays it's very rare to lose the entire Spot
capacity at once, and AutoSpottng will help you mitigate this by falling back to
on-demand instances.

## Which workloads are tolerant to Spot interruptions?

Spot interruptions sometimes happen for a given instance type, so the workloads
running on Spot need to be able to handle them gracefully.

This is often the case for AutoScaling groups, especially if they are running
containerized applications, like it's often the case nowadays, but most
workloads running on AutoScaling groups that scale dynamically or sit behind
HTTP load balancers or consume data from SQS queues are good candicates to Spot,
because new requests can be routed quickly to new instances without any visible
user impact.

## How are interruptiona handled by AutoSpotting?

The interruptions are published in the instance metadata and EventBridge.

AutoSpotting listens for these events and take actions, such as detaching the
instances from the load balancer and terminating them gracefully, giving the
chance to the AutoScaling group to provision on-demand capacity.

## How does AutoSpotting work?

Normally the Spot bidding process is static and can be done manually using the
AWS console, but since AWS exposes pretty much everything via APIs, it can also
be automated against those APIs.

AutoSpotting monitors the AutoScaling groups where it was enabled and it
continuously replaces existing or new on-demand instances found in those groups
with compatible and identically configured Spot instances.

It will be default replace all on-demand instances with Spot but it can also
keep some of them running as on-demand is configured so.

## What are the goals and design principles of AutoSpotting?

AutoSpotting is designed to be used against existing AutoScaling groups with
long-running instances, and it is trying to be as close as possible to
invisible, usually it just does it thing without you noticing anything.

It's also meant to be rolled out at scale, such as across entire AWS
organizations, where it can sometimes be executed in opt-out mode, converting
the entire infrastructure to Spot instances.

The configuration is designed to be minimalist and everything should just work
without much tweaking. You're not expected to need to determine which instance
types are as good as your initial ones, which instance type is the cheapest in a
given availability zone, and so on. Everything should be determined based on the
original instance type using publicly available information and querying the
current Spot prices in real time. Your main job is to make sure your application
can sustain instance failures, and to configure a draining action suitable for
your application and environment if you do need one.

It also tries as much as possible to avoid locking you in, so if you later
decide that Spot instances aren't for you and you want to disable it, you can
easily do it with just a few clicks or commands, and immediately revert your
environment to your initial on-demand setup, unlike most other solutions where
the back-and-forth migration effort may become quite significant. You can even
enable/disable it on a one-off or scheduled basis, if you have such a use case.

From the security perspective, it was carefully configured to use the minimum
set of IAM permissions needed to get its job done, nothing more, nothing less.
There is no cross-accounting IAM role, everything runs from within your AWS
account and no information about your infrastructure ever leaves your AWS
account unless you maybe ship its logs yourself to a CloudTrail S3 bucket.

## What is the use case in which AutoSpotting makes most sense to use?

Any workload which can be quickly drained from soon to be terminated instances.

AutoSpotting is designed to work best with relatively similar-sized, redundant
and somewhat long-running stateless instances in AutoScaling groups, running
workloads easy to transfer or re-do on other nodes in the event of Spot instance
terminations.

Here are some classical examples:

- Development environments where maybe short downtime caused by Spot
  terminations is not an issue even when instances are not drained at all.
- Stateless web server or application server tiers with relatively fast response
  times (less than a minute in average) where draining is easy to ensure
- Batch processing workers taking their jobs from SQS queues, in which the order
  of processing the items is not so important and short delays are acceptable.
- Docker container hosts in ECS, Kubernetes or Swarm clusters.

Note: AutoSpotting implements some termination monitoring and draining logic
which can be extended if you use termination lifecycle hooks.

## What are some use cases in which it's not a good fit and what to use instead?

Anything that doesn't really match the above cases:

### Groups that have no redundancy

If you have a single instance in the group, Spot terminations may often leave
your group without any nodes. If this is a problem, you should not run
AutoSpotting in such groups, but instead use reserved instances, maybe of T2
burstable instance types if your application works well on those.

### Instances which can't be drained quickly

If your application is expected to serve long-running requests, without timing
out after longer than a couple of minutes, AutoSpotting(or any Spot automation)
may not be for you, and you should be running reserved instances.

### Cases in which the order of processing queued items is strict

Spot instance termination may impact such use cases, you should be running them
on on-demand or reserved instances.

### Stateful workloads

AutoSpotting doesn't support stateful workloads out of the box, particularly in
case certain EBS persistent volumes need to be attached to running instances.

The replacement Spot instances will be started but they will fail to attach the
volume at boot because it is still attached to the original instance. Additional
configuration would have to be in place in order to re-attempt the attach
operation a number of times, until the previous on-demand instance is terminated
and the volume can be successfully attached to your Spot instance. The Spot
instance's software configuration may need to be changed in order to accommodate
this EBS volume.

## How do I install it?

You can launch it using the provided CloudFormation stack. It only takes a
couple of minutes and just needs a few clicks in the AWS console or a single
execution of awscli from the command-line.

The same CloudFormation stack template can also be used for launching a StackSet
against your entire AWS organication.

Alternatively, there is also a community-supported Terraform stack which works
similarly, a Docker container image you can run anywhere, and even configuration
on how to run it on Kubernetes as a cronjob.

You can see the Getting Started
[documentation](https://github.com/AutoSpotting/AutoSpotting/blob/master/START.md)
for more information on all these installation methods, as well as the initial
setup procedure.

## In which region should I install it?

For technical reasons, the CloudFormation installation only works with the
US-East region.

The community-supported Terraform stack can be launched in any AWS region.

You only need to install it once, at runtime it will by default connect across
all regions in order to take action against your enabled AutoScaling groups.
This is configurable in case you want to only have it running against a smaller
set of regions.

## How do I enable it?

The entire configuration is based on tags applied on your AutoScaling groups.

By default it runs in `opt-in` mode, so it will only take action against groups
that have the `spot-enabled` tag set to `true`, across all the enabled regions.


It can also be configured to run in `opt-out` mode, so it will run against all
groups except for those tagged with the `spot-enabled` tag set to `false`. This
mode is unique to AutoSpotting, and when combined to the StackSet deployment
methos is a great way to adopt Spot at any scale.

Some large companies are using it in `opt-out` configuration even across
organizations with hundreds of AWS accounts to ensure the majority of their
infrastructure to run on cost-effective Spot instances and migrated to Spot in
very short time, in some cases literally overnight, without requiring any
cooperation and engineering effort from their development teams.

This can be challenging but it has been done so if you are also considering such
a rollout and would like support from someone who did it repeatedly and knows
how to avoid the pitfalls of such a large-scale migration project, feel free to
get in touch on [gitter](https://gitter.im/cristim).

Note: the keys and values of the `opt-in`/`opt-out` tags are configurable in
both modes, and multiple tags can be used.

## What if I have groups in multiple AWS regions?

As mentioned before, the groups can be in any region, AutoSpotting will connect
to all regions unless configured otherwise at installation time.

The region selection can be changed later by updating the CloudFormation or
Terraform stack you used to install it.

## Will it replace all my on-demand instances with Spot instances?

Yes, that's the default behavior (we find it quite safe), but for your peace of
mind this is configurable, as you can see below.

## Can I keep some on-demand instances running just in case?

Yes, you can set an absolute number or a percentage of the total capacity, using
the global configuraton set at install time, or you can override it on a per-AutoScaling group basis using tags set on each group.

For more information about these tags, please refer to the Getting Started
guide.

## What does it mean "compatible"?

The launched Spot instances can be of different type than your initial
instances, unless configured otherwise, but always at least as large as the
initial ones and always cheaper.

You can also constrain it to the same type or a list of types,
also using the main configuration or some optional tag overrides on a per-group
level.

The launched Spot instances have at least as much memory, disk
volumes(both size and their total number), CPU and GPU cores, EBS optimization,
etc. as the initial on-demand instance type.

The price comparison also takes into account the EBS optimization extra fee
charged on some instance types, so if your original instance was EBS optimized
you will still get an EBS optimized instance, and the prices are correctly
compared, taking the EBS optimization surcharge into account.

Often the initial instance type, or the same size but from a different
generation, will be the cheapest so it is quite likely that you will get at
least some instances from the original instance type, but it can also happen
that you get larger Spot instances.

## How is the Spot instance replacement working?

AutoSpotting supports two execution modes: cron(legacy) and event-based and they
usually run in parallel.

The new event-based mode will listen on EventBridge for new instance launch
events. If the instances are launched for an enabled AutoScaling group
(including for newly created groups), the instance launch events are captured
and inserted in a FIFO SQS queue created in your account when installing
AutoSpotting.

AutoSpotting then processes them sequentially from the SQS FIFO queue for a
given group, in order to keep capacity constant. For each new on-demand instance
launch event AutoSpotting will launch a compatible Spot replacement instance.
The Spot instance is immediately attached to the group so that it's bootstrapped
from scratch as part of the group. The on-demand instance is terminated
immediately after the Spot instance was launched, so from the perspective of the
group this is much like the replacement of an unhealthy instance. The
replacements are done sequentially for a group at the rate allowed by the
synchronous attach/detach API calls, also influenced  by the load balancer
connection draining timeout.

AutoSpotting also runs periodically against your enabled groups in the so-called
cron mode, which usually kicks in in the following cases:

- when AutoSpotting has been recently installed and configured against existing
  AutoScaling groups
- when the FIFO queue isn't present
- when the event-based mode is disabled by configuration
- for the rare cases when the configuration has changed to replace more
  on-demand instances with Spot
- when we have to reap on-demand instances that failed to be replaced such as
  when there's not enough Spot capacity or for whatever other reasons such as
  API throttling, API outages, etc.

When running AutoSpotting is running in cron node and notices that more
than configured on-demand instances are running there it will randomly pick an on-demand instance and initiate a replacement by
launching a compatible Spot instance.

The new Spot instance will be tagged with the name of the AutoScaling group for
which it was launched, and the algorithm waits for the Spot instance to be
launched. Once the Spot instance resource was launched and became available
enough for receiving API calls, the instance is tagged to the same tags set on
the initial instance, then the algorithm stops processing instances in that
group. It does this in parallel over all your groups across all AWS regions
where it was configured to run.

On the next run, maybe 5 minutes later, it verifies if the launched Spot
instance was running for enough time and is ready to be added to the group. It
currently considers the grace period interval set on the group and compares it
with the instance's uptime.

If the Spot instance is past its grace period, AutoSpotting will attach it to
the group and immediately detaches and terminates an on-demand instance from the
same availability zone. Note that if draining connection is configured on ELB
then Auto Scaling waits for in-flight requests to complete before detaching the
instance. The terminated on-demand instance is not necessarily the same used
initially, just in case that may have been terminated by some scaling operations
or for failing health checks. But nevertheless, it will be picked from the same
Availability Zone in order to avoid rebalancing actions.

For both modes

- the chosen Spot instance type will be the cheapest with Spot capacity
  available at the time in the same availability zone as the on-demand instance
  selected for replacement.
- the Spot instance is configured identically to the original instance, sharing
  the security groups, SSH key, EBS block device mappings, EBS optimization
  flag, tags, etc. This information is taken mainly from the group's launch
  configuration, which is otherwise kept unchanged and would still launch
  on-demand instances if needed.
- the Spot price is by default set to the hourly price of the original on-demand
  instance type.

## What happens in the event of Spot instance terminations?

AutoSpotting detects and processes both termination notification
events and rebalancing events with the same logic.

The behavior in this case depends on your group's configuration. If you use a
termination Lifecycle Hook, the Spot instance will be terminated immediately by
AutoSpotting, which will trigger the immediate execution of the Lifecycle Hook.
There you can take actions such as pushing log files to S3, draining ECS cluster
tasks, etc. and hopefully manage to drain it completely before it's forcefully
terminated by the Spot market.

If the group has no termination Lifecycle Hook configured, the instance will be
detached from the group and left running until the Spot market will eventuallly
terminate it. This will trigger its automatic removal of the instance from the
load balancer.

Once the instance was terminated, your AutoScaling group's health checks will
eventually fail, and the group will handle this as any instance failure by
launching a new on-demand instance likely in the same availability zone as
initially configured on the group.

That on-demand instance will be replaced later on by AutoSpotting using the
normal replacement process described above.

## How does AutoSpotting compare to the the legacy AutoScaling Spot integration?

Or why would I use AutoSpotting instead of the normal AutoScaling groups which
set a Spot price in the launch configuration or the mixed groups?

The answer is it's more reliable than the legacy Spot AutoScaling groups,
because they are using a fixed instance type so they ignore any better priced
Spot instance types, and they don't fallback to on-demand instances when the
market price is higher than the bid price across all the availability zones, so
the group may be left without any running instances.

AutoSpotting-managed groups will out of the box launch on-demand instances
immediately after Spot instance terminations and AutoSpotting will only try to
replace them with Spot instances when compatible instances are available on the
market at a better price than those on-demand instances. The Spot capacity is
still temporarily decreased during the price surge for a short time, until your
on-demand instances are launched and configured, but the group soon recovers all
its lost instances.

On top of that, AutoSpotting allows you to configure the number or percentage of
Spot instance that you tolerate in your ASG, while the legacy integration of AWS
would try to replace them all, causing potential downtime if they were to
terminate at the same time.

## How does AutoSpotting compare to the AutoScaling mixed groups?

AutoSpotting is much like the relatively new mixed groups feature, but it still
has some unique capabilities:

- automated fail-over to on-demand instances when Spot capacity is unavailable
  and back to Spot soon after the Spot market conditions improve
- you can enable/disable it at will from CI/CD pipelines and even on a schedule
  basis
- you can roll it out across your entire fleet in opt-out mode, without any
  configuration changes, in particular you don't need to convert your groups to
  LaunchTemplates
- flexible/automated instance type selection for Spot instances

To be fair it also does have a few drawbacks (some currently being worked on):

- no support for the AWS regions located in China
- no support for GovCloud
- the small runtime costs
- some software costs if you don't build it from the open source code yourself

## How does AutoSpotting compare to the Spot fleet AWS offering?

Or why would I use AutoSpotting instead of the Spot fleets? And when would I be
better off running Spot fleets?

The Spot fleets are groups of Spot instances of different types, managed much
like AutoScaling groups, but with a different API and configuration mechanism.
Each instance type needs to be explicitly configured with a certain bid price
and weight, so that the group's capacity can be scaled out over various instance
types.

These groups are quite resilient because they are usually spread over multiple
Spot instance types, so it's quite unlikely that the price will surge on all of
them at once. But much like the default AutoScaling Spot mechanism they are also
unable to fall back to on-demand capacity in case the prices surge across all
their instance types. AutoSpotting will also try to avoid using all instances of
the same type, in many cases, with enough capacity by spreading over three or
four different Spot market price graphs, which in addition to the on-demand
fallback capability should be also quite resilient in the event of Spot
terminations.

Also the SpotFleet configuration mechanism is quite complex so it's relatively
hard to migrate to/from them if you already run your application on AutoScaling
groups, which is trivial to do with AutoSpotting.

The SpotFleets are also much less widely used than the AutoScaling groups, and
many other AWS services and third-party applications are integrated out of the
box with AutoScaling but not with SpotFleets. Things like ELB/ALB,
[CodeDeploy](CODEDEPLOY.md), and Beanstalk would run pretty much out of the box
on AutoScaling groups managed by AutoSpotting, while integrating them with
SpotFleets may need additional work or would simply be impossible in their
current implementation. People also tend to be much more familiar with the
AutoScaling group concept, which is easier to grasp and makage by developers
which have more limited exposure with AWS.

Spot Fleets are great for use cases in which the instance type is not important
and can vary widely, or workloads can be somehow scheduled on certain instance
types, like for example in case of ECS clusters.

AutoSpotting, on the other hand will try to keep the instances relatively
consistent with each other, so instances will be in a narrower range than
usually configured on the Spot fleets. This is a consequence of the current
implementation which doesn't have any weighting mechanism, so in order to
meaningfully scale capacity with the same AutoScaling policies, the instances
have to be roughly of the same size. The original on-demand price used for Spot
instance bidding will also constrain the Spot instance types to a relatively
narrow range, which is not the case for SpotFleets.

## How does AutoSpotting compare to commercial offerings such as SpotInst?

Many of these commercial offerings have in common a number of things:

- SaaS model, requiring admin-like privileges and cross-account access to all
  target AWS accounts which usually raises eyebrows from security auditors. They
  can read a lot of information from your AWS account and send it back to the
  vendor and since they are closed source you can't tell how they make use of
  this data. Instead, AutoSpotting is launched within each target account so it
  needs no cross-account permissions, and no data is exported out of your
  account. Also since it's open source you can entirely audit it and see exactly
  what it does with your data and you can tweak it to suit your needs.
- Implement new constructs that mimic existing AWS services and expose them with
  proprietary APIs, such as clones of AutoScaling groups, maybe sometimes
  extended to load balancers, databases and functions, which expect custom
  configuration replicating the initial resources from the AWS offering. Much
  like with Spot fleets, this makes it quite hard and work-intensive to migrate
  towards but also away from them, which is a great vendor lock-in mechanism if
  you're a start-up, but not so nice if you are a user. Many of these resources
  require custom integrations with AWS services, which need to be implemented by
  the vendor. Instead, AutoSpotting's goal is to be invisible, easy to install
  and remove, so there's no vendor lock-in. Under the hood it's all good-old
  AutoScaling, and all its integrations are available out of the box. If you
  need to integrate it with other services you can even do it yourself since
  it's open source.
- they're all pay-as-you-go solutions charging a percentage of the savings. For
  example Spotinst charges 20% or often as much as you will pay AWS for Spot
  instances, which I find obscene for how simple this functionality can be built
  in AutoSpotting. They justify this by nice looking dashboards and buzz words
  such as Machine Learning, but although that's nice to have, it's not really
  needed to implement this type of automation. Predicting the Spot prices is
  hard so it's better to invest the time automating the draining process and
  making it faster to react when terminations happen. Their goal is to sell a
  product which has to look and feel polished enough for people to buy it.
  AutoSpotting's goal is to simply be useful, and as invisible as possible, also
  from the price perspective. If you need to see a saving dashboard, just look
  at the Bills section of the AWS console.
- From the functionality perspective they are indeed more feature-rich and
  polished than both AWS Spot Fleets and AutoSpotting, and they may be
  cloud-provider-agnostic, but their price tag is huge.

## How does AutoSpotting compare to commercial offerings such as Xosphere and MaxSpotter?

Both these commercial tools are based on the older MIT-licensed AutoSpotting
source code with a few additional closed-source extensions. So far none of them
has contributed anything back towards AutoSpotting development and they don't
give any credit in their marketing materials.

They operate similarly to AutoSpotting\s cron mode in many ways, so if you don't
really need their additional features it's relatively easy to migrate amongst
them by renaming the tags set on your AutoScaling groups, or you can even
re-configure/customize AutoSpotting to accept their configuration tags. Please
[get in touch](https://gitter.im/cristim) if you need help migrating from them
to AutoSpotting.

All in all you get a few of the AutoSpotting benefits, but at a much higher cost
overhead. Below you can see a rough feature/cost comparison with both of them.

### Xosphere

Pros

- Offers native Kubernetes and ECS integration
- Supports attaching EBS volumes for stateful workloads
- Available on the AWS Marketplace
- Supports GovCloud
- Automatic diversification of Spot instances
- Has a costs dashboard

Cons

- Lacking most modern AutoSpotting features, especially the event-based
  replacement since they forked the code back in 2018
- Charges 15-20% of savings depending on instance type and pricing model.
- There's no published formula on how to calculate your costs and they reserve
  the rights to change prices at a month notice, so you can't predict your costs
  in advance.

When to choose Xosphere over AutoSpotting

- you depend on their closed source additional functionality. Keep in mind that
  some of them are also doable with AutoSpotting but might require additional
  effort.
- you have infrastructure in GovCloud
- their unpredictable and much higher costs are okay for you

### MaxSpotter

Pros

- User-friendly dashboard
- Uses machine-learning for preemptive shutdown of terminating instances and
  choosing instances with low interruption rates.

Cons

- Lacking some recent developments in AutoSpotting since they forked the code
  back in 2019
- Charges 50% of savings(Professional plan)

When to choose MaxSpotter over AutoSpotting

- you depend on their closed source additional functionality
- their much higher cost overhead is not a concern for you

## How much does it cost me to run it?

AutoSpotting is designed to have minimal footprint, and it will only cost you a
few pennies monthly.

It is based on AWS Lambda, the default configuration is triggering the Lambda
function once every 5 minutes, and most of the time it runs for just a couple of
seconds, just enough to evaluate the current state and notice that no action
needs to be taken.

In case replacement actions are taken it may run for more time because the
synchronous execution of some API calls takes more time, but most of the times
it finishes in less than a minute. It should be well within the monthly Lambda
free tier, you will only pay a few cents for logging and network traffic
performed against AWS API endpoints.

The Cloudwatch logs are by default configured with a 7 days retention period
which should be enough for debugging, but shouldn't cost you so much. If
desired, you can easily configure the log retention and execution frequency to
any other values in the CloudFormation stack parameters.

## How about the software costs?

The software itself is free and open source so there it's entirely free if
you build the open source code straight from trunk, host it on your own
infrastructure and test it all yourself.

But as building it and setting it all up may take some time and effort, we also
have convenient to use evaluation binaries built straight from trunk. These are
free to use, can be installed in minutes but haven't been yet tested thoroughly
and expire after 30 days since being built.

If you don't want to take any chances or spend time testing the trunk builds
yourself, we also have supported prebuilt binaries that have been thoroughly
tested. These cost a small monthly fee but also support further development
of the software.

You therefore have the following options:

### Open source

- the source code is and will always be distributed under an open source
  license, so anyone can build and run their own binaries free of charge on any
  number of AWS accounts.
- users can freely fork and customize the code, but these forks may be hard
  to maintain on the long term if they diverge consistently from mainline.
  Do yourself a favor and at least try to upstream your local changes.
- limited, best-effort community support, and no support whatsoever if you
  run a custom fork with private changes.
- the AutoSpotting name is also a registered trademark covering only the
  use in relation to the official binaries, the source code from the mainline
  git repository and its public forks made in the process of contributing code
  to this software.

### Evaluation binaries

- These are binaries and Docker images built from the git repository after each
  commit.
- They will save you the time needed to build the source code yourself from
  trunk, upload and host it on your own infrastructure.
- Expire after a month from being built, designed to help find issues with the
  latest code.
- These builds may sometimes be broken and barely tested. You will need to test
  the software yourself to make sure it works well from you.
- Limited, best-effort community support, and only if you can reproduce the
  problem with the latest build.

### Stable Proprietary binaries

- Carefully selected binaries that were tested and confirmed to work well, so
  you shouldn't run into any problems.
- They come with installation help, long term enterprise support and
  notifications over a private channel when a newer stable version is available.
  These notifications are only given to the stable build subscribers.
- Licensed in a flat, inexpensive monthly subscription basis payable through
  [Patreon](https://www.patreon.com/join/cristim).

### Exception for contributors

- Code contributors, people contributing testimonials or blogging about the
  project can use the stable binaries free of charge on any number of AWS
  accounts, for a year after their latest contribution to the project.
- You will need to get in touch on gitter and prove that you qualify for this,
  and will be given some special installation instructions.
- These don't include the enterprise support, but the limited, best-effort
  community support of the evaluation binaries.


## Does AutoSpotting continuously search and use cheaper Spot instances?

Or in other words if I attach autoSpotting to an AutoScaling group that is 100%
Spot instances, will it autobid for cheaper compatible ones when found later on?

The answer is No. The current logic won't terminate any running Spot instances
as long as they are running, and since they are using the on-demand price as bid
value they may run for a relatively long time while other cheaper Spot instance
may become available on the market. The only times when AutoSpotting interacts
with your instances is at the beginning, after scaling actions or immediately
after Spot instances are terminated and on-demand instances are launched again
in the group.

## The lambda function was launched but nothing happens. What may cause this?

I have a couple of on demand instances behind an asg configured with the
required tags but still no Spot instance is bieng launched. What is the problem?

Have a look at the logs for more details.

Spot instances may fail to launch for a number of reasons, such as market
conditions that manifest in low capacity across all the compatible instance
types, but also if you run a trunk build there might be bugs. If you are
impacted by such issues please report it on GitHub, try the stable binaries or
consider
[contributing](https://github.com/AutoSpotting/AutoSpotting/blob/master/CONTRIBUTING.md)
a fix.

## Which IAM permissions does AutoSpotting need and why are they needed?

Just like users who pipe curl output into their shell for installing software
should carefully review those installation scripts, users should pay attention
and audit the infrastructure code when launching CloudFormation or Terraform
stacks available on the Internet, especially in case they are given significant
permissions against the AWS infrastructure.

AWS is quite helpful and by default it forbids installation of stacks which have
the potential to be used for escalation of privileges, but it turns out
AutoSpotting needs such permissions in order to work.

In order to launch the AutoSpotting stack, you will need to have admin-like
permissions in the target AWS account and you need to give the stack a special
permission, called `CAPABILITY_IAM`, which is needed because the stack creates
additional IAM resources which could in theory be abused for privilege
escalation. You can read more about this in the official AWS
[documentation](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-iam-template.html#using-iam-capabilities)

The AutoSpotting stack needs this capability in order to create a custom IAM
role that allows the Lambda function to perform all the instance replacement
actions against your instances and autoscaling groups.

This configuration was carefully crafted to contain the minimum amount of
permissions needed for the instance replacement and logging its actions. The
full list can be seen in the Cloudformation stack
[template](https://github.com/cristim/autoSpotting/blob/master/cloudformation/stacks/AutoSpotting/template.yaml#L199),
but it basically boils down to the following:

- describing the resources you have in order to decide what needs to be done
  (things such as regions, instances, Spot prices, existing Spot requests,
  AutoScaling groups, etc.)
- launching Spot instances
- attaching and detaching instances to/from Autoscaling groups
- terminating detached instances
- logging all actions to CloudWatch Logs

In addition to these, for similar privileges escalation concerns, the
AutoSpotting Lambda function's IAM role also needs another special IAM
permission called `iam:passRole`, which is needed in order to be able to clone
the IAM roles used by the on demand instances when launching the replacement
Spot instances. This requirement is also pretty well
[documented](https://aws.amazon.com/blogs/security/granting-permission-to-launch-ec2-instances-with-iam-roles-passrole-permission/)
by AWS.

Since AutoSpotting is open source software, you can audit it and see exactly how
all these capabilities are being used, and if you notice any issues you can
improve it yourself and you are more than welcome to contribute such fixes so
anyone else can benefit from them.

## Is the project going to be discontinued anytime soon?

No way!

The project is actually growing fast in popularity and there are no plans to
discontinue it, actually it's quite the opposite, contributions are accelerating
and the software is already quite mature after more than 5 years of constant
development.

New people install it every day, there are thousands of installations, many
people are using it in production, including on large scale installations with
millions in yearly savings.

The stable binaries are generating recurring income to the initial author, who
is this way incentivized to continue development. This has been growing steadily
over the years, and supports the development of the software. If
you like this project please consider also signing up to a monthly subscription.

## How do I Uninstall it?

You just need to remove the AutoSpotting CloudFormation or Terraform stack.

The groups will eventually revert to the original state once the Spot market
price fluctuations terminate all the Spot instances. In some cases this may take
months, so you can also terminate them immediately, the best way to achieve this
in a controlled manner is by configuring AutoSpotting to use 100% on-demand
capacity for a while before uninstalling it.

Fine-grained control on a per group level can be achieved by removing or setting
the `spot-enabled` tag to any other value. AutoSpotting only touches groups
where this tag is set to `true`.

Note: this is the default tag configuration, but it is configurable so you may
be using different values.

## Shall I contribute to AutoSpotting code?

Of course, all contributions are more than welcome :-)

For details on how to contribute have a look
[here](https://github.com/AutoSpotting/AutoSpotting/blob/master/CONTRIBUTING.md)
