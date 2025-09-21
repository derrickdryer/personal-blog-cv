---
layout: post
title: Rebuilding
lead: Another day, another time I'm rebuilding the lab
---

For those who are unacquainted, rebuilding a home lab is a rite of passage. Whether you’re adding new hardware, implementing a new network architecture, or just chasing the latest experiment, rebuilding is the point of a home lab. At least for a while…

This time around, I wanted to focus on **clarity and separation**. My previous builds taught me that it’s too easy for everything to blur together — so this rebuild is all about making boundaries.

## Production vs. Development

In a typical enterprise environment, workloads are divided into Production and Development. There are others (like staging, QA, etc.), but these two cover the essentials:

- **Production** → Stable, predictable, pinned. Once something is in prod, I want to rely on it daily without surprise.  
- **Development** → Unstable, experimental, breakable. This is where new services get tested before they graduate to prod.  

Bringing this mindset into my lab helps me learn good habits and prevents me from accidentally breaking the services I actually depend on.

## The Old Setup

My old environment had **everything on one VLAN** with no clear separation between stable and experimental workloads.  
- Every VM, LXC, and Pi lived on the same flat network.  
- Hostnames weren’t standardized, so it wasn’t always clear what a system was or whether it was safe to mess with.  
- I had no way of quickly telling what “just worked” versus what was a test bed for some new shiny thing.  

It worked — but it got messy fast.

## Dividing It Up with DNS

This time, I’m standardizing how I identify systems with **DNS records that encode environment**.  

Pattern: `[hostname]-[num].[environment].hidden.it.com`


Examples:  
- `pve-1.prod.hidden.it.com` → Proxmox hypervisor in production  
- `truenas-1.prod.hidden.it.com` → TrueNAS storage in production  
- `rpi-4.dev.hidden.it.com` → Raspberry Pi node in development  

On the k3s cluster, I’ll mirror this logic with **namespaces**:  
- `prod` namespace → Long-lived, stable workloads (DNS, monitoring, storage helpers).  
- `dev` namespace → Playgrounds for Helm charts, beta releases, and experiments.  

This way, the same mental model applies across Proxmox and Kubernetes.

## Why Keep Everything on One VLAN (For Now)

I’m keeping all systems under **VLAN 100** for now.  

Benefits:  
- **Simplicity** → Bring services up quickly without juggling routing or firewall rules.  
- **Visibility** → Nodes see each other without extra hops.  
- **Easier troubleshooting** → When something breaks, I know it’s not my VLAN segmentation at fault.  

Trade-offs:  
- No isolation between prod and dev.  
- Security is weaker than it could be.  
- Scaling this long-term would get messy.  

It’s not perfect, but it keeps the rebuild focused and reduces variables while I iron things out.

## How This Helps in the Long Run

This structure pays off later in a few ways:  
- **Documentation** → Hostnames and namespaces tell their own story. No need to dig into configs to know what’s what.  
- **Consistency** → Proxmox and k3s follow the same prod/dev split, making it easier to reason about deployments.  
- **Future-proofing** → When I move to segmented VLANs, add monitoring, or automate deployments, the foundation is already clean.  

## Next Steps & Lessons Learned

This rebuild is only the beginning. Looking ahead:  
- **Network segmentation** → Separate prod, dev, and storage into their own VLANs.  
- **High availability** → Add redundancy for the hypervisor and storage layer.  
- **Monitoring/alerting** → Prometheus + Grafana for visibility into everything.  
- **CI/CD in k3s** → GitOps-style pipelines for deploying services cleanly.  
- **More automation** → Ansible/Terraform to cut rebuild time in half next time.  

The main lesson: every rebuild is a chance to make the lab a little more intentional. This one lays the groundwork for stability and organization, while still keeping the freedom to experiment.

## Future Post - Kubernetes/k3s

This next step is a big one for me: moving further into **containerization**.  
Up until now, most of my work has been with Docker, but I’m starting to outgrow it. Docker is great for learning and running single containers, but Kubernetes opens the door to managing clusters, scaling workloads, and experimenting with cloud-native patterns.

I’ll be the first to admit: I’m not very experienced with Kubernetes yet. That’s exactly why I’m tackling it — the best way to learn is to build.  

So in a future post, I’ll be covering my next big leap:  

- Building a **k3s cluster** on Raspberry Pi 5 nodes  
- Using **k3sup** to bootstrap and manage it  
- Learning the ropes of Kubernetes by comparing it to what I already know from Docker  
- Sharing what works, what breaks, and what I wish I’d known sooner  

Stay tuned — it’s going to be fun, and probably a little messy. But that’s the point of the lab.