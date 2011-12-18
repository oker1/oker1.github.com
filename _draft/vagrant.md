---
layout: post
title: Trying chef
---
I've wanted to try chef for a while, and I finally did it. I figured the best way to try it would be trying to
replicate a complex setup I've already done with puppet. This way I don't have to figure out scripting and
configuration issues, I can concentrate only on the actual configuration management, all in a well-know domain.

I'm pretty sure my prior knowledge of puppet helped a lot, but taking it into account, I still feel that chef has a
gentle learning curve compared to puppet. The order of the run list is fully deterministic and that makes things much
easier than puppet's dependency declarations. I haven't seen the downside of this explicit ordering yet, but I haven't
done a lot yet, so that might come up later.