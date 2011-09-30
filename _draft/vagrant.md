---
layout: post
title: CentOS with Vagrant and VeeWee
---
Some tips:

 * The prebuilt CentOS boxes have oudated guest additions (4.0), you'll have to build your own using VeeWee
 * The latest release of VeeWee is outdated, ssh won't work in the boxes (github changed urls for accessing files directly)
 * The CentOS templates are outdate in VeeWee
 * You'll have to check out VeeWee from source, both of the above are fixed in master

After finding out all of these, I've managed to build a CentOS 6 basebox.