---
title: "Data-centric environment management"
published: false
date: '13-01-2018 06:00'
taxonomy:
    category:
        - blog
    tag:
        - freckles
        - freckelize
        - way-too-long
author: 'Markus Binsteiner'
toc:
    headinglevel: 3
---

This is something that bugged me for what feels like ever: whenever I have one or a set of files I work with, I need to initially do some work to prepare my laptop or server or container or whatever to deal with those files. For example: 

- if I have Python source files, I need to make sure I have (the right version of) Python installed, a virtualenv created, and dependencies installed via `pip`
- if I have markdown files representing content for (this here) blog, I need to setup and configure a webserver, maybe install PHP and probably also some (the right) PHP libraries, as well as [grav](https://getgrav.org)
- if I have a dump of a database, I need to have the `mysql` (or other) database server installed to be able to use the dump to restore all it's tables, then I have to execute the restore command 

Now, what bugs me is that this is repetitive work, and work that shouldn't be necessary as those files usually already contain all the metadata needed to figure out what they are and what they need in terms of host environment to be useful. And even if not, just augmenting those sets of files with a minimal amount of metadata should be enough to remove any uncertainty. 

What I want is a tool I point at a folder, and which does everything necessary. Without me having to hold hands, provided it can find all the metadata it needs.

So, long story short, I wrote such a tool: [freckelize](https://docs.freckles.io/en/latest/freckelize_command.html). `freckelize` is part of a project called [freckles](https://github.com/makkus/freckles) which is an experiment to find ways to re-use all those existing tasty [Ansible](https://ansible.com) modules and roles for things beside 'traditional' configuration management.

For the purpose of this blog post I will refrain from writing about details how `frecklelize` works, what features beside the basic ones it has, what the security implications of a tool like that are, and anything else that might distract from getting across the basic idea behind it.

So, to get an idea what I'm talking about, a (simple) example:

### preparing a machine to serve a static webpage

---
**NOTE**

As of now, the examples below are only tested on Debian Stretch. Other platforms and distribution versions would be easy to add, but I haven't had the time yet.

---

For the most simple case, all we need is a `index.html` file, containing some minimal html:

```
<!DOCTYPE html>
<html>
<body>
<h1>Now, what is this?</h1>
<p>No idea at all, mate.</p>
</body>
</html>
```

Put that in a folder `my_site`. Now, assuming `freckelize` is already installed, we'd type this:

```
freckelize static-website -f my_site/
```

`freckelize` has so called 'adapters' which deal with certain types of data profiles. The adapter for the static website data profile is called, well, `static-website`. What this adapter will do is:

- installs the `nginx` webserver, to be run as the user who owns the `my_site` folder (as otherwise there might be no read permission -- this can be configured though, see below)
- configures the `nginx` webserver to listen on `localhost` port 80 (which is the adapter default and can be changed)
- configures a virtual host that uses the `my_site` folder as it's root
- makes sure the `nginx` systemd service is enabled and started

### adding metadata

#### adding metadata: type of the data-set

To keep everything neat and tidy, I think it's a good idea to add metadata about the `my_site` folder to the folder itself. `freckelize` by default reads a file called `.freckle`, which uses the [yaml](https://en.wikipedia.org/wiki/YAML) format and sits in the root of the data-set/folder.

---
** NOTE **

For convenience -- I realize and don't care it sounds slightly stupid -- I might refer to the folder containing data that `freckelize` can handle as a `freckle` or a `freckle folder` from now on. 

---

This is what we put in the `.freckle` file to tell `freckelize` the data is a static website:

```
- static-website
```


Among other things, that allows us to let `freckelize` worry about which adapter to use:

```
freckelize -f my_site
```

Notice how we don't use the `static-webpage` command anymore. Also, on a sidenote: `freckelize` uses Ansible as the backend that does the actual work of setting up the environment, and as Ansible runs are (mostly) [idempotent](https://en.wikipedia.org/wiki/Idempotence) we can run those commands as often as we want without breaking things.

#### adding metadata: the port the webserver should listen on

Now, let's assume port 80 is not a good port to use. Maybe we already have another webserver running and this is only for development. Or we are using this inside a Vagrant box that only forwards port 8080. Doesn't matter. Here's how we change the `.freckle` file to use port 8080:

```
- static-website:
    static_website_port: 8080
```

After another `freckelize -f my_site` we can visit [http://127.0.0.1:8080](http://127.0.0.1:8080) in our browser and should be able to get to our shiny page.




I had this idea. Imagine you have:

- a folder containing metadata about the type of data it contains, as well as the data itself
- a generic (configuration-management-like) tool that knows how to 'read' said folder and that provides a library of actions commonly needed to setup machines
- an adapter (basically a plugin for that tool) for each type of data you are interested in, those adapters know how to bring a machine from an unknown state to one that enables the machine to work with said data

If you had that, in order to setup a machine to be able to work with that data, you'd have to:

- install the tool, if not already installed, as well as any extra adapters you might need
- put the folder with your data on your machine
- point the tool to the folder, and wait for it to bring your machine to a state that is able to support the data

Let's use a static webpage as a simple example. The data would look something like:

- index.html

In addition to that, we'd have a metadata file that describes the type of data we are dealing with, as well as (optional configuration options, if applicable):

static-webpage:
  webserver: nginx
  
In order to process that, we'd have an adapter called 'static-webpage' that would install either the Apache webserver, or nginx, depending on configuration provided in the folder metadata.

Now, before the obligatry 'we-already-can-do-that-with-our-current-tools-why-would-we-need-something-new-i-don't-like-new-ideas-and-potential-change'-reaction kicks in, let me say that, yes, this is obviously not useful or a good idea in every case where one needs to setup one or a set of machines in a reproducible way. But I have an intuition that a workflow like this could simplify tooling and it's user interface as well as speed up configuration in certain cases.

What would be different? 

Well, for one, by storing configuration and other metadata 'physically' with the data a service or application uses we don't have to worry about having and maintaining a place for our infrastructure configuration. No registry, or 'inventory'. We have to backup our data anyway, so why not also store everything else related to that data with it?

Then, we could build up a repository of data types along with adapters that could process those data types. Those adapters could be developed and improved over time, for example to support different Linux distributions for the (data) host system, even different OSs. As a bonus side-effect, such a repository would also serve as a place that would have information about best practices and good layouts for each of the

