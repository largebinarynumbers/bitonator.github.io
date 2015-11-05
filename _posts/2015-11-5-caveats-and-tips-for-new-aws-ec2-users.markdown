---
layout: post
title:  "Caveats and Tips for New Amazon EC2 Users"
date:   2015-05-21 22:05:44
categories: projects
permalink: /modular-project-environments-with-chroot/
excerpt: some gotchas and some tips to help you get around Amazon AWS services easily
---

Instances
=========

1. Be aware of the region in which you create your instance because storage etc. cannot be assigned across regions
2. Open up the ports that you need by defining a security group for your instance. 
3. The warning that all hosts will be able to access your ports can be ignored for public servers.
4. Backup your private key, you can't get it back if you lose it.
6. You can assign the same key to multiple instances, super convenient.
7. Name your instances, things get messy very fast, as you start spinning up more instances.
8. Its safe to reboot your instance from the command line
9. When you reboot from the command line, your IP address remains the same, if you do it from the AWS console, then its a new IP address for the same server, baby
10. Setup swap space to prevent your server from becoming inaccessible when it runs out of memory, EBS swap is fine now that IOPS are bundled with storage costs.
11. Make OS images from the instance menu, not from snapshots, because those are not HVM complaint and won't run on all types of instances.
12. Instance is too small for your needs? Have no fear, you can stop, change instance type, start and voila! great new hardware.
13. Consider reserved instances once your setup is finalized, to reduce costs.
14. Stopping an instance is similar to powering down, Terminating an instance is similar to throwing it in the junkyard(time to junk the instance included)
15. Open only ports that you need, our fully open server was hacked within days.
16. Modifying /etc/fstab is a great way to lock yourself out of your instance, use rc.local instead to mount volumes.

Storage
=======

1. EBS is cheap as hell, don't worry much about creating volumes or snapshots
2. If you have an EBS Storage of 100 GB and you use only 5 GB, you will be billed for only 5GB, so feel free to allocate more than enough storage.
3. Snapshots are incremental
4. Snapshot in pending status? No worries, you can still modify your data while AWS backs it up(it snapshots a copy of your volume)
5. Avoid making OS images from snapshots, they aren't HVM enabled and won't work on all types of instances.
6. You can create volumes from snapshots easily.
7. Volumes can be detached and attached to any instance at any time.
8. Attached EBS volumes show up as `/dev/sdf, /dev/sdg, ...` or `/dev/xvdf,/dev/xvdg,...` depending on the linux flavour
9. Snapshots and volumes in a different region? sucks, you'll have to create a new volume in the region you want it in. Be aware of this when you make snapshots and volumes.
10. S3 Uploads from an EC2 instance are much faster than from say, your machine.
11. S3 downloads are very fast, I didn't really use cloudfront on a audio streaming site, and things were just fine.
12. From 11, think a bit before using Cloudfront, the additional headache of managing cached copies may not be worth it. 
13. S3 can be configured with CORS to serve AJAX content from your website.

Miscellaneous
=============

1. Public IP addresses are cool, use 'em. Makes switching instances while maintaining the same IP address easy.
2. Avoid using third-party AWS clients. Everything you have is critical to your business or project. Giving access to a third-party client is as safe as giving your office keys to your competitor.
3. Chances are you'll end up paying for compute power and not storage.

My Production Setup
===================

Here's how I've configured my instances and storage, in case it helps.

1. Everything is in Oregon west-2a region. I check this when creating instances, volumes and snapshots.
2. I have a dev server and a production server, with two different keys
3. both instances have a swap space of twice the RAM allocated for the instance.
4. data volumes are created separately and mounted on the instances
5. I use a bind mount `mount -o bind`, which allows me to mount a folder from the data volume to a folder in the root filesystem.
6. I have an OS image created out of the configured production server, so that I can spin up a new server quickly.
7. Having data on a separate volume means I can terminate a buggy instance, start a new instance, mount the data volume, and voila! i'm back up.
8. I have one public IP allocated to the production instance.
9. I take snapshots of my data volume everyday.
