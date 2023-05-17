---
title: "From Dust We Come"
date: 2023-04-23
draft: true
---

Any good validator should be running a robust architecture - in a real datacenter with security, redundant power and networking, yada yada.  Or better yet across multiple real datacenters for higher reliability.  That can blow up the opex though.  Real validators should also be running some form of the sentry node architecture to defend against ddos attacks.  Sentry node architecture, however, literally implies many active nodes at all times (more than the bare minimum of one), multiplying your capex and opex. 

But we of humble beginnings remain of humble beginnings.  And as such the goal is to minimize all exes. Therefore we made the seemingly rational decision to move all operations to our own in-house datacenters, the proverbial "basement" (the literal basement).  We will run a local HA kubernetes cluster comprised of industrial (consumer) grade mini PCs.  We will have monitoring and automation in place to activate backup nodes in the cloud when somethimg bad happens locally and recovery is not imminent. Such as a power outage, a kid pouring water on a node, or something stupid that a staff member (me) does.  We're anxiously awaiting to see how this goes.  

## Why kubernetes?

I like it and it's what I know is all.  aka there is no good reason. In fact there are alot of good reasons NOT to use kubernetes, namely the complexity introduced.  These are highly stateful applications after all... 

But, if I had to search for a good reason to do this:
 
- Makes the hardware cattle.  In theory, given sufficient spare capacity, we should be able to handle node failures seamlessly
- Thus scales better too.  Adding capacity is plug and play
- Standardize deployment procedures in very mature and active tooling projects (helm, ansible, docker)
- Easier to pick up and move to another location one day (wishful thinking)
- Chase the tech.  Kubernetes is the biggest open source project ever for a reason.
- Maybe become akash provider

## The Design

- Our local cluster will have an HA control plane, and since this isn't the cloud, we will use keepalived for sharing a local vip for the api server via this role:  https://github.com/RnodeC/RnodeC.keepalived
- rke2 is the distro of choice for its OOTB security and quality. Here is the ansible role we are using to setup our rke2 cluster:  https://github.com/RnodeC/RnodeC.rke2
- All nodes will be connected to a vpn in order to protect my home IP.  Hopefully, in the event of a ddos-attack, my vpn provider will be able to offer some protection.
- kube-prometheus-stack installed
- clamscan anti-virus scanning the nodes daily
- I did install Longhorn storage, and would love to use it, but it just seemed way too chatty and slow for the nature of these blockchain applications.  For now, we had to settle on plain old hostPath storage.  

## The Validators

We'll be migrating our eth2 validator over here once all systems are go.  We use geth as our execution engine and lighthouse as the consensus client.  Both have worked flawlessly on bare metal.  In kubernetes, so far so good.  Here are the helm charts used for these:  https://github.com/RnodeC/helm-charts/tree/main/charts/geth and https://github.com/RnodeC/helm-charts/tree/main/charts/lighthouse

Other validators (bera) will be here soon.  Need to monitor these for awhile and see how the cluster behaves.  

## The Battles

Nothing is ever easy, and the hard parts are generally unpredictable.  In designing this whole system, here were the major 
 
- selinux:  0% chance we would run this without selinux, but of course selinux = battles.  Chatgpt ftw.
- keepalived script:  figuring out how to tell keepalived to not participate in the consensus until the rke2 service was ready was really tricky for some reason, but was not an option to not have this working.
- nginx tcp/udp services:  this was the biggest battle of all.  Took alot of troubleshooting and help from various communities, but eventually learned that, basically, ingress-nginx was not going to be able to proxy the tcp/udp services.  The downside of this is that now we must forward ports directly to specific nodes where I know my applications are running.  Between this and the storage situation, basically what we have now is the same thing as a bare metal setup.  Gotta roll over before you can crawl though.
- decisions... what pc to buy, what to do with storage, reverse proxy in the cloud?  private vpn server in the cloud?  