---
layout: post
title: Referencing base box files in Vagrantfile 
---
I was trying to create a vagrant box for a development environment, which can be easily distributed and used by other developers. I've created a base box using debian squeeze, and made a vagrant box based on it. I've added puppet provisioning to it, changed the ssh key, etc. I had a fair amount of configuration in the Vagrantfile, which shouldn't be changed by the users. I found out that one can include the Vagrantfile, and other files in the base box, and the Vagrantfile's values will take precedence over the box's own Vagrantfile.


One problem I ran into, and couldn't figure out from vagrant's documentation was referencing the files included in the base box. With some googling, I found an [issue](https://github.com/mitchellh/vagrant/issues/344) on GitHub with this exact same question.