---
published: true
layout: post
author: MegaWubs
categories: Docker
---

In my previous post about Docker I wrote that my setup was far from ideal. This week that changed radically. I’ve finally found the way to develop with Docker on my mac, no full blown VM involved!

After weeks and weeks of searching, trial and error I gave it up last week. The solution I had backfired on me and I went all in on the Linux Mint virtual machine installed on my Mac. No more issues with mounted volumes or sftp/rsync syncing from mac to the VM. Just everything directly in the VM, even PHPStorm. Luckily I got coherence working so it wasn’t a complete pain in the ass. 

I worked like this for one day. It didn’t work.

My ideal of Docker on OS X is that it Just Works. No VMs I have to configure, no hoops I have to go to, no weird tools I have to completely understand in order to make it work. I just want to open my terminal and be able to run docker-compose up without first having to log into the Linux vm that runs Docker. I also want to work inside OS X, I have a Mac for a reason. Also, I need it to be fast.

### Docker Machine NFS
Did I say I gave up? Well apparently not, this week I found the solution. Reading trough GitHub issue after issue hidden in the corners of GitHub, I found a tool called [docker-machine-nfs](https://github.com/adlogix/docker-machine-nfs). You run it against your Docker Machine of choice once and it Just Works!

How it works? It removes the terribly slow `/Users` vboxshare in your docker machine VM and replaces it with an NFS share your mac provides. This way, all Docker volumes can be created from your local system through the docker machine VM to the docker container. It’s true that I still need a VM, but this is the basic boot2docker VM that’s part of the Docker ecosystem. It’s way different from the full blown Linux Mint VM I used previously. 

Now, I can work completely from my Mac. No Linux Mint involved. I can even work with PHPStorm normally without it complaining that it’s working from a remote folder, or getting stuck on indexing the remote folder. 

And did I mention that it’s fast? Like, amazingly vast? The web application I develop is now even faster compared to working form inside the Linux Mint VM. Where it first had a page load of 200 to 600 ms, it now has a page load of 90 ms or less. This is a huge speedup compared to vboxshare’s 1.5 seconds load time.
