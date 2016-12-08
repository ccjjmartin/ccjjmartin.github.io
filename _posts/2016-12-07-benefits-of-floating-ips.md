---
layout: post
title: "Benefits of a Floating (Virtual) IP"
description: "Quicker site launches, reliability, minimal downtime."
tags: [server admin]
categories: [intro]
time: 5 minutes
image:
  background: triangular.png
---

You might have heard about high availability before but didn't think your site
was large enough to handle the extra architecture or overhead.  I would like to
encourage you to think again and be creative.

<!-- more -->

## Background:

Digital Ocean has a concept they call a [floating IPs](https://www.digitalocean.com/company/blog/floating-ips-start-architecting-your-applications-for-high-availability/).  This idea is great, it allows you to keep your site running
in the event of failure.

## Credit

I have to give credit to BlackMesh for handling this process quite well.  The
only thing I had to do was create the tickets to change the architecture and
BlackMesh implemented it.

## Exact Problem:

One of our support clients had the need for a complete site relaunch due to a
major overhaul in the underlying architecture of their code. Specifically, they
had the following changes:

1. Change in the site docroot

2. Migration from a single site architecture to a multisite architecture based
on domain access

3. Upgrade of PHP version that required a server replacement/upgrade in linux
distribution version

Any of these individually could have benefited from this approach.  We just
bundled all of the changes together to delivering minimal downtime to the sites
users.

## Solution

So, what is the right solution for a data migration that takes well over 3 hours
to run?  Site downtime for hours during peak traffic is unacceptable.  So, the
answer we came up with was to use a floating IP that can easily change the
backend server when we are ready to flip the switch.  This allows us to migrate
our data on a new separate server using it's own database (essentially having
two live servers at the same time).

![Example Diagram](/images/posts/2016-12-07-benefits-of-floating-ip-example-diagram.svg)

## Benefits:

Notice that we won't need to change the DNS records here which meant we didn't
have to wait for DNS records to propagate all over the internet.  The new site
was live instantly.

## Additional details on the transition

Some other notes during the transition that may lead to separate blog posts:

1. We created a shell script to handle the actual deployment and tested it before
the actual "go live" date to minimize surprises.

2. A private network was created to allow the servers to communicate to each
other directly and behind the scenes.

3. To complicate this process, during development (prelaunch) the user base grew
so much we had to off load the Solr server on to another machine to reduce
server CPU usage.  This means that additional backend servers were also involved
in this transition.

## Go live (migration complete)

After you have completed your deployment process, you are ready to switch the
floating ip to the new server.  In our case we were using "keepalived" which
responds to a health check on the server.  Our health check was a simple php
file that responded with the text true or false.  So, when we were ready to
switch we just changed the health checks response to false.  Then we got an
instant switch from the old server to the new server with minimal interruption.

![Example Diagram](/images/posts/2016-12-07-benefits-of-floating-ip-example-diagram-new.svg)

## Acceptable losses of this approach

There were a few things we couldn't get around:

1. The need for a content freeze

2. The need for a user registration freeze

The reason for this was that our database was the database updates required the
site to be in maintenance mode while being performed.

A problem worth mentioning:

1. The database did have a few tables that would have to have acceptable losses.
The users sessions table and cache_form table both were out of sync when we
switched over.  So, any new sessions and saved forms were unfortunately lost
during this process.  The result is that users would have to log in again and
fill out forms that weren't submitted.  In the rare event that a user changed
their name or other fields on their preferences page those changes would be
lost.

## Additional considerations

1. Our mail preferences are handled by third parties

2. Comments aren't allowed on this site
