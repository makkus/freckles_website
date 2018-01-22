---
title: "Data-centric environment management"
published: true
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

I hate repetition. And I especially hate repetitive work. Unfortunately -- probably because of that whole thing with the apple 6000 years ago -- I always find myself doing repetitive work. Even when I always always always go out of my way to avoid doing repetitive work. Often I find myself writing scripts which take me 10 times longer to write than doing the repetitive thing I try to avoid a million-billion times by hand. 

If you are working in I.T., chances are you hate repetitive work also. Luckily, a lot of I.T. work is devoted to cutting down on repetitive work. Because, after all, that's what computers are good at: give them the same problem several times, and they'll work on it the same way all those several times. Without complaining, mostly. Even if they have to do it repetitively a million-billion times and then a million-billion times again.

Besides any potential personal aversion against repetitive work one might have there is another reason humans shouldn't be doing repetitive work if a computer can do it: the chance of making a mistake grows the more often you do a repetitive thing. Computers either do it always wrong, or -- better -- always right. Provided, of course, somebody wrote tests for all possible and impossible edge cases. Which, obviously, always happens.

===

So, long story short, there is one (non-obvious, but very generic) repetitive thing that, over the years, annoyed me more than other repetitive things: setting up an environment on a computer (virtual, physical, whatever) in order for this computer to deal with a set of files/data that is of a type it isn't prepared to deal with yet. 

For example: 

- if I have Python source files, I need to make sure I have (the right version of) Python installed, a virtualenv created, and dependencies installed via `pip`
- if I have markdown files representing content for (this here) blog, I need to setup and configure a webserver (say, `nginx`), maybe install PHP and probably also some (the right) PHP libraries, then I have to download [grav](https://getgrav.org) and put it into the right folder so `nginx` can find it
- if I have backup data of a service that needs migration to a new machine (virtual or not) I have to re-setup and configure the service, and restore the backup data in some way or other

It appears to me that, if I know what type of data I deal with, and if I have a set of (ideally best) practices for that type of data, a computer would be able to do that sort of setting up for me. Right? ... RIGHT???

Not only that, if the computer would know what kind of platform/distribution/version of distribution it runs (which it always does since, hey, it's the one running it...), and if it had instructions that outline what to do differently an each of those platforms, it could always, automatically, prepare a host environment that is hospitable to the kind of data in question, and there would be no manual intervention necessary. AT ALL.

What would be necessary is somebody preparing those sort of recipes, best practices and platform-dependent instructions for all the potential types of data we come across. In a way that the computer can understand. But, the good thing is, we could do that in a collaborative and evolutionary fashion, starting off with a simple use-case, and build on top of that to support more options, features, and platforms in the future. We'd have one place to improve a recipe for a give use-case or type of data, and that recipe would go through the normal stages of software development until it can be considered stable and comprehensive enough. 

So, what's left is the glue, an application that runs on the computer, is pointed at the data we are interested in, parses that data (and potential augmenting metadata), chooses the right recipes for that type of data and platform it runs on, and executes those recipes in the way the data/metadata demands.

There are two types of existing applications that do parts of what I'm describing: configuration management engines like [Ansible](https://ansible.com), [SaltStack](https://saltstack.com/), [Puppet](https://puppet.com), etc., and build systems like [make](https://www.gnu.org/software/make/), [maven](https://maven.apache.org/), [Rake](https://github.com/ruby/rake) and so on. But those are either focused on a bigger infrastructure and network environment, only understand a certain type of data (Java project, Ruby project, ...), or are very low-level and don't have the building blocks to manipulate the state of a machine in an efficient way. There might be other tools, but if there are, I don't know about them.

Now, all of this led me to work on [freckelize](https://docs.freckles.io/en/latest/freckelize_command.html). `freckelize` is part of a project called [freckles](https://github.com/makkus/freckles) which is designed as a layer on top of 'Ansible', and is an experiment to find ways to re-use all those existing tasty [Ansible](https://ansible.com) [modules](http://docs.ansible.com/list_of_all_modules.html) and [roles](https://galaxy.ansible.com/) for things beside 'traditional' configuration management.

In this blog post I'm not going into too much detail about how `freckelize` works, what features beside the basic ones it has, what the security implications of using a tool like that are. And anything else that might distract from getting across the basic idea behind it. 

To illustrate that basic idea, I'll use the very simple example of hosting a static webpage, where the data we work with is a simple, and single, html file. I also wrote a more in-depth blog post about this usage scenario, so if you are interested in those details, check it out [here](XXXX).

---

**NOTE**: the 'static-website' recipe I'm using below is currently only tested on Debian Stretch

---

The example dataset -- a single html file, plus an optional metadata file named `.freckle` -- can be found here: [https://github.com/freckles-io/example-static-webpage](https://github.com/freckles-io/example-static-webpage).

Let's put those two files in a folder called `example-simple-website`. The `index.html` file looks like this:

```html
<!DOCTYPE html>
<html>
<body>
<h1>Now, what is this?</h1>
<p>No idea at all, mate.</p>
</body>
</html>
```

And this is `.freckle` file, which contains additional metadata:

```yml
- static-website:
    static_website_domain: 127.0.0.1   # ip address or domain name used by this server
    static_website_port: 80            # port the webserver listens to
```

This latter `.freckle` file is optional, but useful to adjust some of `freckelize`'s behavior. It uses `yaml` syntax, and contains a list of types of data to be considered, including potential variables per type. In this case it contains two variables, which both are set to default values, which means that this file doesn't affect behavior just yet.

So, this is what you have to do (assuming `freckles` is already installed) to install a webserver (`nginx` in this case) and configure it to host our website:

```
freckelize static-website -f example-simple-website
```

Done. Check if it's working by visiting: [http://127.0.0.1](http://127.0.0.1)

Since this folder already contains a '.freckle' file that includes the 'static-website' item, we could have just omitted the 'static-website' command:

```
freckelize -f example-simple-website
```

Should we not have that folder on our local machine but only on Github, we can let `freckelize` also clone it for us:

```
freckelize -f https://github.com/freckles-io/example-static-webpage.git -t /var/lib/freckles
```

This will clone the repository as a sub-folder of `/var/lib/freckles` (which is a nice place to collect those sort of folders). Then it'll do the same things it did before using the local folder.

There are more scenarios `freckelize` supports, like for example pointing it to a remote tarball of the data. Refer to [the documentation](https://docs.freckles.io) for details. In the future, anyway. Need to re-write parts of that documentation to bring it up-to-date. Sorry.

As an example this is not really impressive, I'm sure, as this is something that would not take a lot of time to do by hand. Just a `sudo apt-get install nginx`, and some configuration editing somewhere in `/etc/nginx/`.

To illustrate how easy it is to accomplish more complex tasks, let's say we want to host that website on a VPS somewhere, via https and a (valid) [Let's encrypt](https://letsencrypt.org/) certificate. This is supported by the `static-website` ([source](XXX))recipe ('adapter' in `freckelize`-speak). We need to provide a bit more information to `freckelize` though, as it wouldn't know the domain name to use, and the email address the folks over at "Let's encrypt" require. Also, we need to configure DNS so that the domain name we use points to the VPS IP address. This has to be done manually, and since it depends a lot on the providers that are used I won't write about how to do that.

Let's edit the `.freckle` file:

```
- freckle:
    owner: www-data
    group: www-data
    
- static-website:
    static_website_domain: example.frkl.io
    static_website_port: 80
    lets_encrypt_email: makkus@frkl.io
```

We leave the port as 80, the adapter will automatically create a vhost configuration to forward all traffic to the default https port (443). The adapter is written in a way that, if it encounters the `lets_encrypt_email` variable with a string other than 'none', it'll use that value as email address and request a https certificate for the domain specified from "Let's encrypt". In addition, it'll setup a cron job that makes sure that certificate will be re-newed before it expires.


Now, there are a lot of reasons why doing 'data-centric environment management' is a stupid idea, and I trust the internet (especially the part that really likes how things are done at the moment) will come up with all of them. And I agree that there are a lot of use-cases where doing it this way is not appropriate at all. I do also think though that there's a niche for a tool like this (doesn't need to be `freckelize` -- happy to see someone write something better), and I especially think it'd be good for all of us if we came together to work on the sort of recipes I was describing above. In a more comprehensive, focused and collaborative way than, for example, Ansible roles are developed at the moment. Creating a repository of 'best practices' for different types data structures along the way...

I'll write more in-detail examples for using `freckelize` in different scenarios, and with different data profiles. So, if you are intersted, check back here every now and then.

Also, please get in touch if you have questions, or suggestions. Either via [email](mailto:makkus@posteo.de), [gitter](https://gitter.im/freckles-io/Lobby), or a [Github issue](https://github.com/makkus/freckles/issues).
