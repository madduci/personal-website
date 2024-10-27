+++
title = "The year of Linux Desktop"
date = 2024-10-27
ref = "the-year-of-linux-desktop"
tags = ["personal", "blog"]
categories = ["thoughts"]
+++

I've done it. After 3.5 years, I've finally asked my manager the permission to change the operating system on my company-issued laptop, I've put Ubuntu 24.04 LTS on the machine and it was a blast, not because I am not aware of the capabilities of Linux, since I'm a Linux User myself since a long time now, but because I was productive two-three hours after the initial setup of the system, including the configuration of the corporate VPN connection and I was able to join the meetings.

# A short introduction

The laptop is a HP ZBook G6 with a Core i9 9th-Gen, 64 GB RAM, two SSD (!) and an nVidia Quadro K1000, shipped with Windows 10. The laptop can manage huge workloads and never had a problem with it, but a couple of things annoyed me _a lot_: the continuous hearing of CPU fan and the constantly saturating the available amount of RAM on the machine.

At my work at gematik, I am on the team responsible for the [DEMIS](https://www.rki.de/DE/Content/Infekt/IfSG/DEMIS/DEMIS_node.html) Project, the "Deutsches Elek­tro­ni­sches Melde- und Infor­ma­tions­system für den Infek­tions­schutz", a Software Solution, directed by the Robert-Koch-Institute and developed by gematik. I'vem ainly helped transforming the back-then on-premise Components (a Service-Oriented Architecture have been already used to split the capabilities of the system across many components) to Containers being able to be executed in a Kubernetes environment.

My typical local development environment is a local Kubernetes Cluster, built with [kind](https://kind.sigs.k8s.io/), an IDE with the opened projects and a browser with 10-to-15 opened tabs. The Kubernetes Cluster, being run though kind in Docker as Containers, made the WSL2 pretty wild in terms of amount of RAM being used.

Tuning the `.wslconfig` file made the experience better, but as long as the DEMIS Environment was steady growing and new components were being developed and deployed, the machine was suffering a lot: even in Idle, you could hear the CPU fan spinning up by some Windows background process and I've got the impression that I was continuously hitting the throttling zone, due to high temperatures of the CPU itself. It was time for a change.

# Planning the move

For the Operating-System-Switch Task, I had only one shot and one day, our internal monthly "Innovation Friday", where developers can learn or try new solutions, or simply sharp their tools.

My plan initially was to install Debian 12, because of its stability in terms of packages and upgrades, plus a stable lifecycle for the kernel being used. An initial test with a Live-USB Stick failed, since my external displays, connected through the Display Port with the docking station, weren't detected at all. An hack is available, but I didn't want to spend a lot of time dealing with it, since a more important theme was ahead of me: VPN.

I've opted for Ubuntu 24.04 since it's an LTS version and offers a pretty much extended support for a lot of hardware. My guess was right, since the external Displays got recognized at the first boot, so we had a candidate for the setup. I've tested also Manjaro Linux, based on Arch, but the SecureBoot on my laptop pretty much disliked it and won't boot it.

# Getting things up 

The typical graphical installation of Ubuntu went straightforward and the configuration of LUKS was also simple. It was pretty wild to discover that I could setup LUKS only with the automatic partitioning tool and not without the manual one, since I have 2 SSDs in the machine, this got me kind of off-guard and I am still asking myself what was the reasoning behind this choice by Canonical.

I got the SecureBoot configuration done pretty fast, since it is very well documented online. What surprised me a lot, was how fast I could setup the corporate VPN. With a small script and relying on `openconnect`, I could join my company network and the establishment of the connection is actually much faster on Linux than on Windows.

What about kind? I was able to setup the whole cluster pretty fast too and - surprise, surprise - the total amount of RAM usage, replicating the one I've had on Windows, is consistently using ~30GB of RAM, compared to the 50-55 required under Windows/WSL2.

A nice side-effect that I've got with Ubuntu is that the CPU fan has been quieter and actually I can hear it only if I spin up a new local Kubernetes cluster or I start the PC, then never again.

# Not everything is gold

Besides all the good parts, which I am thankful for, and that made me really productive since Day-0, there are some drawbacks in running Linux, especially if your company is heavily Microsoft-oriented. Being Microsoft Teams the go-to Meeting Application, means that you have to use the Progressive Web App (PWA), which works only under Chromium and not under Firefox with adblockers and Strict mode enabled.

The PWA itself works good, even the screen/monitor sharing feature, but was made it more complicated was the lack of any filter/gain in Audio/Video quality: my camera keeps showing me dark and the audio presents artifacts, because there's no noise-suppression.

Interestingly enough, the Camera and the Microphone are fully functional outside Teams, so it means that the quality is extremely degradated only under Chromium.

# Conclusion

I am running Ubuntu now since a month and it has been so far successful. I felt myself more productive, due to less interruptions (forced updates anyone?) and feeling generally safer, in terms of lower attack surface than in Windows. A major drawback is represented by the quality of the Teams PWA, because of the missing Audio/Video enhancements, but I still can be heard and seen decently.

In short, after using Linux on my private laptop and installing it on my working one, I can finally say it: the 2024 was the year of Linux Desktop for me!