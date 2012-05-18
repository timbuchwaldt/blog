---
layout: post
title: "Backup Strategy"
date: 2012-05-18 18:20
comments: true
categories: 
---
Lets talk backup! I just realised my strategy was not as good as it could have been, and I decided to improve it in various spots.
My main issue was the lack of off-site backups, and to accomodate that, I chose Amazon S3. On my Macs I'm running Arq, which works like a charm. I do hourly backups of the most valuable data, using up to 50 GB of S3 Storage (currently using about 3GB). I use Arq on all of my Macs now, in addition to TimeMachine backup to my FreeBSD home server. The only machine left unbackuped was my Ubuntu server that hosts this page (and a few others, e.g. customer sites). To backup this machine I use duplicity, which is a script capable of uploading encrypted, differential backups to S3.

To include the MySQL data from me & my customers, I added automysqlbackup, which does daily, weekly and monthly snapshots from my SQL tables, which are then also included in the S3 backups. Automysqlbackup also sends me e-mails when anything goes wrong backing up, which I consider a really great feature. Additionally, the server is backed up daily by my hoster (Host Europe), which increases the security even further.