---
title: 'So, I made this ..thing'
published: false
date: '17-10-2017 00:00'
taxonomy:
    category:
        - blog
    tag:
        - freckles
        - ansible
author: 'Markus Binsteiner'
toc:
    headinglevel: 3
---

I call it ['***freckles***'](https://github.com/makkus/freckles).

It was one of those 'scratch-your-own-itch'-situations, I suppose. Only, I really didn't want to scratch that particular itch myself.

===

[TOC]
For one, because I knew exactly how long it'd take me to write this if I wanted to do it properly. And I really thought it is something so obvious that sooner or later somebody will build (something like) it, and all I have to do is go on Reddit, moan a bit about how it's bloated, and (obviously) written in the wrong language. But I'll use it anyway, you can thank me later. 

It looks like I was wrong though, and it's either not obvious at all, or other people were playing the same waiting game as me. Or of course it is just a really stupid idea. Not counting that out yet. Either way, I waited for a few years, and nobody scratched my itch. So, eventually I gave in and wrote that ...'thing'. I named the thing 'freckles', and although it's not quite finished yet, I consider it usable. It's out now on [github](https://github.com/makkus/freckles).

What am I talking about? What thing?? Glad you asked! 

## The thing: (easy) 'local' configuration management

### freckles

## Why should I be interested?

That depends. If you like to write your own bash scripts and configure every little nook and cranny on your machine manually, you most likely won't like *freckles*. I for one don't like to do those things, I just want things to be the way I want them to be without me having to do anything, ideally. Or, if that is not an option (which seems to be the case disturbingly often), with as little input (aka, 'work') from my side as possible. 

As I've mentioned above, configuration management is usually used if the environment it is applied to is deemed more important than the time/money it takes to set up and configure everything manually (or the money a potential longer-ish downtime costs). *freckles* tries to change that equation by making it easier, and faster, to practice configuration management, at least in local development environments. I do think there's a lot of developers time to be saved, to be used on actual development, rather than all the annoying stuff around it. Plus, a bit of good practice never hurt anybody, right?

*freckles* is designed to do configuration management for single machines. Physical or virtual, local or remote. Where you have 'root' access, or you don't. Any type of box you want to get into a certain state. Unlike other configuration management systems, it doesn't need any infrastructure because everything runs on the box that needs the state change. So, it's (obviously) not a good fit for medium or large infrastructures (even though I could imagine it play a small part in managing those as well (might write another blog-post about that later).

### Blah-blah-blah. What exactly does this do?

I'll not go into detail here how exactly it works and how to use it, I've written quite a bit of documentation over at [https://docs.freckles.io](https://docs.freckles.io). Check that out if you are interested. But I'll give you a quick overview, and a few examples what *freckles* is good for, below.

I've created three (command-line) frontends for *freckles*: `freckelize`, `frecklecute` and `freckles` itself. The first one i former, you point to a (remote or local) piece of data or code, then it uses the (inherent and/or explicit) metadata of that piece of data to determine what to do to your machine's state. Sorta 'data-centric' configuration management. 

The second, `frecklecute`, is more traditional; you provide it with a list of (state-changing) tasks (basically Ansible modules and roles), and it executes those tasks successively. The third one lets you script `freckelize` and `frecklecute` runs and execute them in one go.

All that does nothing you couldn't do with just Ansible. It's just a bit more convenient.

Part of that convenience is that all three applications can be bootstrapped and executed via the *inaugurate* script (which can also be found on [github](https://github.com/makkus/inaugurate)). I initially wrote that one for *freckles*, but decided to make into it's own project. This allows you to run any of the two frontend cli applications without any requirements installed (except for either `curl` or `wget`) or other preparation work to be done on the machine you want to execute it on. There are (obviously) some security considerations you have to take into account. I've written a bit about that and how *inaugurate* works [here](https://github.com/makkus/inaugurate#how-does-this-work-what-does-it-do), [here](https://github.com/makkus/inaugurate#is-this-secure), and [here](https://docs.freckles.io/en/latest/bootstrap.html). And about trust in general [here](https://docs.freckles.io/en/latest/trust.html).

Basically, the first time you run `freckles` (or `freckelize` or `frecklecute` instead), you do it like:

```
curl https://freckles.io | bash -s -- freckles <freckles_arguments>
```

From that point on-wards, *freckles* (along with it's three command-line interfaces) is installed (in your home directory, under `$HOME/.local/inaugurate` -- a folder you can easily delete if you get tired of *freckles*, without affecting the rest of your system), and you can just call it normally. As you would any other application (you have to re-login for it's PATH to be picked up though): 

```
freckelize <freckelize_arguments>
# or
frecklecute <frecklecute_arguments>
# or
freckles <freckles_arguments>
```

Below are a few examples/suggestions of use-cases where *freckles* makes sense. Roughly, *freckles* is not useful for when you need to do 'actual' work, like running a model to cure cancer. It can however, help you to get started with 'actual' work, by setting up the environment such work needs (e.g. installing applications, dependencies, downloading data repositories, ...). 

Examples:

- after installing a new (physical or virtual) machine, quickly install and configure it with the applications you commonly use, by letting *freckelize* download and process a remote (or local) dotfile/configuration repository (more info: [here](https://docs.freckles.io/en/latest/adapters/dotfiles.html))
- setup the source code of the projects you are working on, including their dependencies, on your machine (e.g. [Python projects](https://docs.freckles.io/en/latest/adapters/python-dev.html), or (generic) [projects that use Vagrant](https://docs.freckles.io/en/latest/adapters/vagrant-dev.html))
- quickly write a short script to install/update some of your own (non-packaged) applications (e.g. *freckles* uses `frecklecute` to [update itself](https://github.com/makkus/freckles/blob/master/freckles/external/frecklecutables/update))
- ensure you and your team-mates use the same setup of a certain development environment
- quickly [execute a 'one-off' Ansible task or role](https://docs.freckles.io/en/latest/frecklecutables/ansible-task.html) (e.g. to install and configure [Docker](https://galaxy.ansible.com/mongrelion/docker/), or [nginx](https://galaxy.ansible.com/geerlingguy/nginx/), etc.), without having to install Ansible itself manually (more info: [here](https://docs.freckles.io/en/latest/frecklecutables/ansible-task.html))
- write easy to read and understand deployment scripts, which can also be used for documentation or education purposes (e.g. in a blog post), and which can be used in combination with *inaugurate* to create 'no-requirement' bootstrap scripts
- create scriptlets that are easy to share and execute, to init new (development or other) projects from templates
- ...


Basically, most things you can imagine which change the state of your machine/filesystem from a relatively 'useless', to a more 'useful' state. The definition of 'useless' and 'useful' is up to you, of course.

I'll probably write more about *freckles* -- what still needs to be done (a lot), why it is like it is, and does the things it does, they way it does them -- in a few more blog posts on this site. Probably. Maybe probably.
