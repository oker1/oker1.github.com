---
layout: post
title: Trying chef
---
I've wanted to try chef for a while, and I finally did it. I figured the best way to try it would be implementing a
complex setup I've already done with puppet. This way I don't have to figure out scripting and
configuration issues, I can concentrate only on the actual configuration management, in a well-know domain.

I'm pretty sure my prior knowledge of puppet helped a lot, but taking it into account, I still feel that chef has a
gentle learning curve compared to puppet. The order of the run list is fully deterministic and that makes things much
easier than puppet's dependency declarations. I haven't seen the downside of this explicit ordering yet, but I haven't
done a lot yet, so that might come up later.

The official chef cookbooks seem really useful, although I've liked writing puppet modules from the ground up,
implementing only what I need. I could do the same thing with chef, but one can go really fast with these cookbooks, so
I don't consider it a pro or con either.

The [base repository](https://github.com/opscode/chef-repo) is useful with the bundled rake tasks, although one missing
piece is the converting of roles from ruby to json. This is needed if you want to use roles written in ruby instead of
json, and you don't use a chef server. I've easily found this written as a [rake task](https://gist.github.com/834890)
that I could append to the Rakefile.

Knife is nice tool, it makes downloading and installing the official cookbooks easy, and you can create cookbook
skeletons with it for your own cookbooks.

One issue I have ran into is the new [dependency based init system](http://wiki.debian.org/LSBInitScripts/DependencyBasedBoot)
in debian squeeze. Chef is not supporting it properly (see [ticket](http://tickets.opscode.com/browse/CHEF-2034)), and
using the official mysql cookbook the mysql server is not started on boot.