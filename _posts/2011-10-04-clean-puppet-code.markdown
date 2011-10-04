---
layout: post
title: Clean Puppet code
---
Puppet is a programming language, and like all others, it's only up to you how well you can use it
to your advantage. It is a very specific, declarative DSL, but I found that most of the principles
of software development still apply to it pretty well.

You have to structure your code otherwise it will become tangled and hard to maintain. Puppet
provides a great feature for this, modules. A module is a directory in the puppet module path.
It has a simple structure, classes and defines are autoimported if you put them in separate manifest files.

Puppet has features for abstractions too. Classes and parameterized classes are great for resources
that can have only one instance on a server. For resources with multiple instances, you can use defines
(custom resources).

To manage dependencies, you can just arrow your code to death, but you'll regret that later, when you
want to refactor your resources. Defining the minimal dependencies explicitly gives you the freedom to
move your resources around, and still maintain the proper ordering between them.

Once you get familiar with all these features, you can use them to create better code. Let's consider
the following example, a simple manifest managing apache2 and two virtual hosts:

<!--
{% highlight puppet %}
class mysite {
	package {
		"apache2":
			ensure => latest,
	} -> service {
		"apache2":
			ensure => running,
	} -> file {
		"/etc/apache2/sites-available/mysite.com":
			content => template("mysite.com.erb"),
			notify  => Service["apache2"],
	} -> file {
		"/etc/apache2/sites-available/admin.mysite.com":
			content => template("admin.mysite.com.erb"),
			notify  => Service["apache2"],
	}
}
{% endhighlight %}
-->
<div class='highlight'><pre><span class='kd'>class</span> <span class='nc'>mysite</span> <span class='p'>{</span>
	<span class='nc'>package</span> <span class='p'>{</span>
		<span class='s'>&quot;apache2&quot;</span><span class='p'>:</span>
			<span class='na'>ensure</span> <span class='o'>=&gt;</span> <span class='s'>latest</span><span class='p'>,</span>
	<span class='p'>}</span> <span class='o'>-&gt;</span> <span class='nc'>service</span> <span class='p'>{</span>
		<span class='s'>&quot;apache2&quot;</span><span class='p'>:</span>
			<span class='na'>ensure</span> <span class='o'>=&gt;</span> <span class='s'>running</span><span class='p'>,</span>
	<span class='p'>}</span> <span class='o'>-&gt;</span> <span class='nc'>file</span> <span class='p'>{</span>
		<span class='s'>&quot;/etc/apache2/sites-available/mysite.com&quot;</span><span class='p'>:</span>
			<span class='na'>content</span> <span class='o'>=&gt;</span> <span class='nf'>template</span><span class='p'>(</span><span class='s'>&quot;mysite.com.erb&quot;</span><span class='p'>),</span>
			<span class='na'>notify</span>  <span class='o'>=&gt;</span> <span class='nn'>Service</span><span class='p'>[</span><span class='s'>&quot;apache2&quot;</span><span class='p'>],</span>
	<span class='p'>}</span> <span class='o'>-&gt;</span> <span class='nc'>file</span> <span class='p'>{</span>
		<span class='s'>&quot;/etc/apache2/sites-available/admin.mysite.com&quot;</span><span class='p'>:</span>
			<span class='na'>content</span> <span class='o'>=&gt;</span> <span class='nf'>template</span><span class='p'>(</span><span class='s'>&quot;admin.mysite.com.erb&quot;</span><span class='p'>),</span>
			<span class='na'>notify</span>  <span class='o'>=&gt;</span> <span class='nn'>Service</span><span class='p'>[</span><span class='s'>&quot;apache2&quot;</span><span class='p'>],</span>
	<span class='p'>}</span>
<span class='p'>}</span>
</pre>
</div>

It's easy to see the biggest problem with this class is that the general apache2 parts can not be
reused without the virtual hosts. We can easily split the class in two:

<!--
{% highlight puppet %}
class apache2 {
	package {
		"apache2":
			ensure => latest,
	} -> service {
		"apache2":
			ensure => running,
	}
}

class mysite {
	file {
		"/etc/apache2/sites-available/mysite.com":
			content => template("mysite.com.erb"),
			require => Package["apache2"],
			notify  => Service["apache2"],
	}

	file {
		"/etc/apache2/sites-available/admin.mysite.com":
			content => template("admin.mysite.com.erb"),
			require => Package["apache2"],
			notify  => Service["apache2"],
	}
}
{% endhighlight %}
-->
<div class='highlight'><pre><span class='kd'>class</span> <span class='nc'>apache2</span> <span class='p'>{</span>
	<span class='nc'>package</span> <span class='p'>{</span>
		<span class='s'>&quot;apache2&quot;</span><span class='p'>:</span>
			<span class='na'>ensure</span> <span class='o'>=&gt;</span> <span class='s'>latest</span><span class='p'>,</span>
	<span class='p'>}</span> <span class='o'>-&gt;</span> <span class='nc'>service</span> <span class='p'>{</span>
		<span class='s'>&quot;apache2&quot;</span><span class='p'>:</span>
			<span class='na'>ensure</span> <span class='o'>=&gt;</span> <span class='s'>running</span><span class='p'>,</span>
	<span class='p'>}</span>
<span class='p'>}</span>

<span class='kd'>class</span> <span class='nc'>mysite</span> <span class='p'>{</span>
	<span class='nc'>file</span> <span class='p'>{</span>
		<span class='s'>&quot;/etc/apache2/sites-available/mysite.com&quot;</span><span class='p'>:</span>
			<span class='na'>content</span> <span class='o'>=&gt;</span> <span class='nf'>template</span><span class='p'>(</span><span class='s'>&quot;mysite.com.erb&quot;</span><span class='p'>),</span>
			<span class='na'>require</span> <span class='o'>=&gt;</span> <span class='nn'>Package</span><span class='p'>[</span><span class='s'>&quot;apache2&quot;</span><span class='p'>],</span>
			<span class='na'>notify</span>  <span class='o'>=&gt;</span> <span class='nn'>Service</span><span class='p'>[</span><span class='s'>&quot;apache2&quot;</span><span class='p'>],</span>
	<span class='p'>}</span>

	<span class='nc'>file</span> <span class='p'>{</span>
		<span class='s'>&quot;/etc/apache2/sites-available/admin.mysite.com&quot;</span><span class='p'>:</span>
			<span class='na'>content</span> <span class='o'>=&gt;</span> <span class='nf'>template</span><span class='p'>(</span><span class='s'>&quot;admin.mysite.com.erb&quot;</span><span class='p'>),</span>
			<span class='na'>require</span> <span class='o'>=&gt;</span> <span class='nn'>Package</span><span class='p'>[</span><span class='s'>&quot;apache2&quot;</span><span class='p'>],</span>
			<span class='na'>notify</span>  <span class='o'>=&gt;</span> <span class='nn'>Service</span><span class='p'>[</span><span class='s'>&quot;apache2&quot;</span><span class='p'>],</span>
	<span class='p'>}</span>
<span class='p'>}</span>
</pre>
</div>

This code seems okay at first, but it can lead to scattered **Package\["apache2"\]** and **Service\["apache2"\]**
dependencies everywhere in your code. The other modules don't have to know that apache2 is a single
package and service, both with the name "apache2". Let's abstract these:

<!--
{% highlight puppet %}
class apache2::package {
	package {
		"apache2":
			ensure => latest,
	}
}

class apache2::service {
	service {
		"apache2":
			ensure => running,
			require => Class["apache2::package"],
	}
}

class apache2 {
	include apache2::package
	include apache2::service
}

class mysite {
	file {
		"/etc/apache2/sites-available/mysite.com":
			content => template("mysite.com.erb"),
			require => Class["apache2::package"],
			notify  => Class["apache2::service"],
	}

	file {
		"/etc/apache2/sites-available/admin.mysite.com":
			content => template("admin.mysite.com.erb"),
			require => Class["apache2::package"],
			notify  => Class["apache2::service"],
	}
}
{% endhighlight %}
-->
<div class='highlight'><pre><span class='kd'>class</span> <span class='nc'>apache2::package</span> <span class='p'>{</span>
	<span class='nc'>package</span> <span class='p'>{</span>
		<span class='s'>&quot;apache2&quot;</span><span class='p'>:</span>
			<span class='na'>ensure</span> <span class='o'>=&gt;</span> <span class='s'>latest</span><span class='p'>,</span>
	<span class='p'>}</span>
<span class='p'>}</span>

<span class='kd'>class</span> <span class='nc'>apache2::service</span> <span class='p'>{</span>
	<span class='nc'>service</span> <span class='p'>{</span>
		<span class='s'>&quot;apache2&quot;</span><span class='p'>:</span>
			<span class='na'>ensure</span> <span class='o'>=&gt;</span> <span class='s'>running</span><span class='p'>,</span>
			<span class='na'>require</span> <span class='o'>=&gt;</span> <span class='nn'>Class</span><span class='p'>[</span><span class='s'>&quot;apache2::package&quot;</span><span class='p'>],</span>
	<span class='p'>}</span>
<span class='p'>}</span>

<span class='kd'>class</span> <span class='nc'>apache2</span> <span class='p'>{</span>
	<span class='kn'>include</span> <span class='nc'>apache2::package</span>
	<span class='kn'>include</span> <span class='nc'>apache2::service</span>
<span class='p'>}</span>

<span class='kd'>class</span> <span class='nc'>mysite</span> <span class='p'>{</span>
	<span class='nc'>file</span> <span class='p'>{</span>
		<span class='s'>&quot;/etc/apache2/sites-available/mysite.com&quot;</span><span class='p'>:</span>
			<span class='na'>content</span> <span class='o'>=&gt;</span> <span class='nf'>template</span><span class='p'>(</span><span class='s'>&quot;mysite.com.erb&quot;</span><span class='p'>),</span>
			<span class='na'>require</span> <span class='o'>=&gt;</span> <span class='nn'>Class</span><span class='p'>[</span><span class='s'>&quot;apache2::package&quot;</span><span class='p'>],</span>
			<span class='na'>notify</span>  <span class='o'>=&gt;</span> <span class='nn'>Class</span><span class='p'>[</span><span class='s'>&quot;apache2::service&quot;</span><span class='p'>],</span>
	<span class='p'>}</span>

	<span class='nc'>file</span> <span class='p'>{</span>
		<span class='s'>&quot;/etc/apache2/sites-available/admin.mysite.com&quot;</span><span class='p'>:</span>
			<span class='na'>content</span> <span class='o'>=&gt;</span> <span class='nf'>template</span><span class='p'>(</span><span class='s'>&quot;admin.mysite.com.erb&quot;</span><span class='p'>),</span>
			<span class='na'>require</span> <span class='o'>=&gt;</span> <span class='nn'>Class</span><span class='p'>[</span><span class='s'>&quot;apache2::package&quot;</span><span class='p'>],</span>
			<span class='na'>notify</span>  <span class='o'>=&gt;</span> <span class='nn'>Class</span><span class='p'>[</span><span class='s'>&quot;apache2::service&quot;</span><span class='p'>],</span>
	<span class='p'>}</span>
<span class='p'>}</span>
</pre>
</div>

This way, the specifics of installing and running apache2 are contained in two classes, and we can
use these as dependencies, without coupling ourselves to the details. This structure gives us an
extension point: If we need an additional package to run apache2, we can add it to **apache2::package**,
without touching any of the resources depending on apache2 being installed. Additionally, it abstracts
the name of these resources from the rest of the code. Of course we could use the name attribute to give
these resources an alias, but this is cleaner, and more flexible.

Now the only weak point is the mysite class. It has duplication, which can be avoided easily, by
introducing a custom resource:

<!--
{% highlight puppet %}
define apache2::vhostfile($content) {
	file {
		"/etc/apache2/sites-available/${name}":
			content => $content,
			require => Class["apache2::package"],
			notify  => Class["apache2::service"],
	}
}

class mysite {
	apache2::vhostfile {
		"mysite.com":
			content => template("mysite.com.erb"),
	}

	apache2::vhostfile {
		"admin.mysite.com":
			content => template("admin.mysite.com.erb"),
	}
}
{% endhighlight %}
-->
<div class='highlight'><pre><span class='kd'>define</span> <span class='nc'>apache2::vhostfile</span><span class='p'>(</span><span class='nv'>$content</span><span class='p'>)</span> <span class='p'>{</span>
	<span class='nc'>file</span> <span class='p'>{</span>
		<span class='s'>&quot;/etc/apache2/sites-available/</span><span class='nv'>${name}</span><span class='s'>&quot;</span><span class='p'>:</span>
			<span class='na'>content</span> <span class='o'>=&gt;</span> <span class='nv'>$content</span><span class='p'>,</span>
			<span class='na'>require</span> <span class='o'>=&gt;</span> <span class='nn'>Class</span><span class='p'>[</span><span class='s'>&quot;apache2::package&quot;</span><span class='p'>],</span>
			<span class='na'>notify</span>  <span class='o'>=&gt;</span> <span class='nn'>Class</span><span class='p'>[</span><span class='s'>&quot;apache2::service&quot;</span><span class='p'>],</span>
	<span class='p'>}</span>
<span class='p'>}</span>

<span class='kd'>class</span> <span class='nc'>mysite</span> <span class='p'>{</span>
	<span class='nc'>apache2::vhostfile</span> <span class='p'>{</span>
		<span class='s'>&quot;mysite.com&quot;</span><span class='p'>:</span>
			<span class='na'>content</span> <span class='o'>=&gt;</span> <span class='nf'>template</span><span class='p'>(</span><span class='s'>&quot;mysite.com.erb&quot;</span><span class='p'>),</span>
	<span class='p'>}</span>

	<span class='nc'>apache2::vhostfile</span> <span class='p'>{</span>
		<span class='s'>&quot;admin.mysite.com&quot;</span><span class='p'>:</span>
			<span class='na'>content</span> <span class='o'>=&gt;</span> <span class='nf'>template</span><span class='p'>(</span><span class='s'>&quot;admin.mysite.com.erb&quot;</span><span class='p'>),</span>
	<span class='p'>}</span>
<span class='p'>}</span>
</pre>
</div>

It's easy to see why this structure is better:

 * Abstracts the location of the vhost files, which is really useful if you want to support multiple
distributions with different directory structures.
 * Abstracts the dependency on the package and the notification of the service so you don't have to
remember to drag those along everywhere you declare a virtual host.