---
title: "freckelize: static website"
published: true
date: '22-01-2018 06:00'
taxonomy:
    category:
        - blog
    tag:
        - freckles
        - freckelize
        - static website
author: 'Markus Binsteiner'
toc:
    headinglevel: 3
---

Let's have a look how to use [`freckelize`](https://docs.freckles.io/en/latest/freckelize_command.html) to setup a machine in order for it to server a static webpage.

===

---
** NOTE **

For convenience -- I realize and don't care it sounds slightly stupid -- I might refer to the folder containing data that `freckelize` can handle as a `freckle` or a `freckle folder` from now on. 

---


### Requirements

- Debian Stretch (other platforms not yet supported, but planned)
- [installed `freckles` package](https://docs.freckles.io/en/latest/bootstrap.html)


### Data

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

#### adding metadata: everything else

To get a list of supported variables for an adapter, use the `freckfreckfreck` command:

```
$ freckfreckfreck adapter-doc static-website

static-website
--------------

  desc: installs and configures webserver to server a static website
  path: /repos/testing/adapters/web/static-website/static-website.adapter.freckle
  role dependencies:
    - geerlingguy.nginx
    - thefinn93.letsencrypt
  available vars:
    static_website_port: the port the webserver should listen on, defaults to 80
    static_website_domain: the domain the static website is hosted on, defaults to '127.0.0.1'
    letsencrypt_email: the email address to use when requesting a https certificate from the 'letsencrypt'-project, defaults to 'none'. if no email address is specified, no https cert will be requested and https won't be setup. Otherwise the static_website_port will be forwarded to the https port (443).

  documentation

    Installs and configures a webserver to publish a static website.
    
    'nginx' is used as the webserver, 'apache' might be supported as an option later. If you set the 'letsencrypt_email' variable this adapter will request a https certificate for the domain set in 'static_website_domain', as well as a cron job to renew it before it runs out. So, for this to work you'll obviously need to have configured dns correctly for the server this is running on.
    
    Supported:
    - for now, only Debian Stretch is supported
    
```


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

