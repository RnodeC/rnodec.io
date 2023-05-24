---
title: "From Dust We Come"
date: 2023-04-23
draft: false
---

Any good validator should be running a robust architecture - in a "real" datacenter with security, redundant power, networking, yada yada.  Or better yet, across multiple datacenters for higher reliability.  That can blow up the opex though.  Validators should also be running some form of the sentry node architecture to defend against DDoS attacks.  Sentry node architecture, however, literally implies many active nodes at all times (more than the bare minimum of one), multiplying your capex and opex.

But we of humble beginnings remain of humble beginnings.  And as such the goal is to minimize all exes. So maybe there is some middle ground to be found between "five 9s" and more of a "best effort".  Why not focus more on recovering from outages than preventing them?  Shouldn't robust networks be designed to survive some validator downtime?  Of course the goal is no downtime, no slashing, but compromises must be made if we want to do this.  Therefore we made the seemingly rational decision to move all operations to our own in-house datacenters, the proverbial "basement" (the literal basement).  We will run a local HA kubernetes cluster comprised of industrial (consumer) grade mini PCs.  We're anxiously awaiting to see how this goes.

## Why kubernetes?

We like it and it's what we know is all... aka there is no good reason. In fact there are a lot of good reasons NOT to use kubernetes, namely the complexity introduced.  These are highly stateful applications after all...

But, if we had to search for a good reason to do this:

- Makes the hardware cattle.  In theory, given sufficient spare capacity, we should be able to handle node failures seamlessly
- Thus scales better too.  Adding capacity is plug and play
- Standardize deployment procedures in very mature and active tooling projects (helm, ansible, docker)
- Easier to pick up and move to another location one day (wishful thinking)
- Chase the tech.  Kubernetes is the biggest open source project ever for a reason
- Opens the door to become an Akash provider

## The Design

- Our local cluster will have an HA control plane, and since this isn't the cloud, we will use keepalived for sharing a local vip for the API server via this role:  https://github.com/RnodeC/RnodeC.keepalived
- rke2 is the distro of choice for its OOTB security and quality. Here is the ansible role we are using to setup our rke2 cluster:  https://github.com/RnodeC/RnodeC.rke2
- All nodes will be connected to a VPN in order to protect our home IP.  Hopefully, in the event of a DDoS attack, my VPN provider will be able to offer some protection.
- kube-prometheus-stack installed.  Will leverage alertmanager to detect any issues with the infrastructure as early as possible.
- clamscan anti-virus scanning the nodes daily.  selinux enabled.
- We did install Longhorn storage, and would love to use it, but it just seemed way too chatty and slow for the nature of these blockchain applications.  For now, we had to settle on plain old hostPath storage.

#### Handling Failures

If something happens - such as a power outage, a hardware failure, a kid pouring water on a node, or something stupid that a staff member (me) does - and a particular validator is knocked offline, we will have monitoring in place to detect the outage, and automation in place to create a node in a public cloud.  No keys will be moved automatically, but the idea is to get a node up and sync-ing asap.  And then it is up to our operations team (also me) to address the failure and bring the original validator back to life, or, if recovery is not imminent, go ahead and manually migrate the validator keys to the new node.  It is important to note that key handling will never be automated.

Here is an incomplete example of how this might be accomplished:  https://github.com/RnodeC/auto-bera-gcp

## The Validators

We'll be migrating our eth2 validator over here once all systems are go.  We use geth as our execution engine and lighthouse as the consensus client.  Both have worked flawlessly on "bare metal."  In kubernetes, so far so good.  Here are the helm charts used for these:  https://github.com/RnodeC/helm-charts/tree/main/charts/geth and https://github.com/RnodeC/helm-charts/tree/main/charts/lighthouse

Other validators (bera) will be here soon.  Need to monitor these for awhile and see how the cluster behaves.

## The Battles

Nothing is ever easy, and the hard parts are generally unpredictable.  In designing this whole system, one of the major trains we got hit by that took longer than one would've thought was figuring out how to get traffic to the cluster.  The goal was to use nginx's TCP/UDP services feature to proxy traffic into the cluster.  Took alot of troubleshooting and help from various communities, but eventually learned that, basically, ingress-nginx was not going to be able to handle this type of traffic pattern (high volume of long-lived UDP sessions).  The downside of this is that now we must forward ports directly to specific nodes where we know our applications are running.  But, it is working.

Between that and the storage situation, basically what we have now is the same thing as a bare metal setup.  Gotta roll over before you can crawl though.
