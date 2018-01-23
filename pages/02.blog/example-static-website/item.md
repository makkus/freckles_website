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

As promised in [this post](/blog/data-centric-environment-management), here is a more in-depth article about how to use [`freckelize`](https://docs.freckles.io/en/latest/freckelize_command.html) to setup a machine in order for it to serve a static webpage.

===

---
** NOTE **

For convenience -- and as a convention -- I might refer to the folder containing data that `freckelize` can handle as a `freckle` or a `freckle folder` in the following.

---


## Requirements

Currently, only Debian Stretch is supported as a host system platform for this as I haven't tested it on anything else yet. Ubuntu will probably work also, but might not. This should change in the future, as the plan is for every adapter like this one to support as many platforms as possible. 

Also, obviously, [the `freckles` package has to be installed](https://docs.freckles.io/en/latest/bootstrap.html) (or you use the 'inaugurate' way of running `freckelize`'s first invocation) in order for this to work.


## Data

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

Let's put that in a folder called `my_site`. 

## Start `freckelize`-ing

Now, assuming `freckelize` is already installed, we'd type this:

```
freckelize static-website -f my_site/
```

`freckelize` has so called 'adapters' which deal with certain types of data profiles. The adapter for the static website data profile is called, well, `static-website`, and you can find it's source [here](https://github.com/freckles-io/adapters/tree/master/web/static-website). What this adapter will do is:

- installs the `nginx` webserver, to be run as the user who owns the `my_site` folder (as otherwise there might be no read permission -- this can be configured though, see below)
- configures the `nginx` webserver to listen on `localhost` port 80 (which is the adapter default and can be changed)
- configures a virtual host that uses the `my_site` folder as it's root
- makes sure the `nginx` systemd service is enabled and started

### Adding metadata

#### Metadata: type of the 'freckle'

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

#### Metadata: the port the webserver should listen on

Now, let's assume port 80 is not a good port to use. Maybe we already have another webserver running and this is only for development. Or we are using this inside a Vagrant box that only forwards port 8080. Doesn't matter. Here's how we change the `.freckle` file to use port 8080:

```
- static-website:
    static_website_port: 8080
```

After another `freckelize -f my_site` we can visit [http://127.0.0.1:8080](http://127.0.0.1:8080) in our browser and should be able to get to our shiny new page.

#### Metadata: generic folder properties

There are a few properties that apply to all freckle folders. The most important ones being `owner` and `group`. Since those are applicable independent of the adapter that is used, they go in their own section of the `.freckle` file:

```
- freckle:
    owner: www-data
    group: www-data
    
- static-website:
    static_website_port: 8080
```

Before this, `freckelize` and the `static-website` adapter used the onwer of the `my_site` folder as the user to run `nginx`. Now, with this new configuration, after re-running `freckelize -f my_site`, the folder will be owned by the `www-data` user, and `nginx` will be run under that same user. As it should be -- apart from when you do development -- as then it's easier to just run the web-server under your own username, so both you and the server have easy read/write permissions on the folder in question.


#### Metadata: everything else

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


#### Option:  adding a "let's encrypt"-certificate

So, according to this, a full-blown, 'production'-ready configuration (minus security hardening, but who needs that anyway...) would look something like:

```
- freckles:
    owner: www-data
    group: www-data
    
- static-website:
    static_website_port: 80
    static_website_domain: example.frkl.io
    letsencrypt_email: makkus@frkl.io
```

We leave the port as 80, the adapter will automatically create a vhost configuration to forward all traffic to the default https port (443). The adapter is written in a way that, if it encounters the `lets_encrypt_email` variable with a string other than 'none', it'll use that value as email address and request a https certificate for the domain specified from "Let's encrypt". In addition, it'll setup a cron job that makes sure that certificate will be re-newed before it expires.

That's it for now, folks. More to come soon, stay tuned for the same thing again, in 'Wordpress'...
