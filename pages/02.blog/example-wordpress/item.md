---
title: "freckelize: static website"
published: false
date: '25-01-2018 18:00'
taxonomy:
    category:
        - blog
    tag:
        - freckles
        - freckelize
        - wordpress
author: 'Markus Binsteiner'
toc:
    headinglevel: 3
---

Another example on how to use [`freckelize`](https://docs.freckles.io/en/latest/freckelize_command.html) to easily setup an environment using only the (relevant) data of an application. This time we will setup a [Wordpress](https://wordpress.com) instance, both locally and later, on a public VPS.

This article focuses quite a bit more on how `freckelize` and an adapter works. Which I reckon will make the reading a bit more difficult and harder to understand. Because of that, I'd like to point out that just using this thing will be quite a bit easier. It's only going to be one command (as I'll show at the end of this post).

Also, for this article, I'll assume you've already read [the one about the `static-website`-adapter](/blog/example-static-website).

===

---
** NOTE **

For convenience -- and as a convention -- I might, below, refer to the folder containing data that is used by `freckelize` as a 'freckle' or a 'freckle folder'.

---

## Requirements / Limitations

Currently, only Debian Stretch is supported as a host system platform for this as I haven't tested it on anything else yet. Ubuntu will probably work also, but might not. This should change in the future, as the plan is for every adapter like this one to support as many platforms as possible. 

Also, obviously, [the `freckles` package has to be installed](https://docs.freckles.io/en/latest/bootstrap.html) (or you use the 'inaugurate' way of running `freckelize`'s first invocation) in order for this to work.

I wrote this freckelize adapter really only as a proof of concept, because I figure it's something a lot of folks know, and have worked with. So it might be a opportunity to showcase what `freckelize` can do. I don't use Wordpress myself, so it might very well be that not everything works as expected.
Also, particularly for this adapter, there are a lot of features that could be added, like for example the option to install additional PHP packages for certain Wordpress plugins. As `freckelize` is really designed to enable the collaborative and easy development of those adapters, I hope that I'll find some people who are interested enough to continue working on this adapter, and make it proper and comprehensively useful.

Lastly, I'll be using [`stow`](https://www.gnu.org/software/stow/) to symbolically link some of the data into place. I want it to be clear that this adapter could have been implemented differently, in a few different ways. This is how I chose to do it because it was the best way I could come up with, and I think it's a good solution. I'm happy to be shown at a better way though. The nice thing with those `freckelize` adapters is, there can be 2 which are implemented totally differently, but which can achieve the exact same thing.

## Data

The `wordpress` adapter is a bit more involved than the `static-website` I wrote about [here](/blog/example-static-website). The following list is the minimal amount of data needed to be able to re-create a Wordpress instance from another one:

- the MySQL database used (well, only the table(s) used by the particular instance)
- the file `wp-config.php` in the web-root
- the contents of the folder `wp-content` in the web-root

We don't want to have the whole 'Wordpress' application in our freckle folder, as that can easily be re-created by downloading the latest version from the Wordpress website. It'd be redundant and would only take up space. We want to separate data from the application, after all. Which is why I chose to keep the 'important' data separate from the rest.

`freckelize`-ing MySQL is not straight-forward either. For the purpose of showcasing `stow`, I decided to do it the way I'll be telling you. Again, it could have been done a number of different ways (for example, by using a mysql dump file to import/export the data into a freckle folder, and creating a cron-job which does the exporting automatically).

### MySQL

So, let's get to it, and let's start with MySQL. The data we are interested in all lives in `/var/lib/mysql`. In order to keep that separate and easy for `freckelize` to access, we create a folder `/var/lib/freckles/wordpress/mysql`. Within that folder, we create another folder called `mysql`, and the `.freckle` metadata file I wrote about before. This is how the latter looks like:

```
- freckle:
   owner: mysql
   group: mysql
   staging_method: stow
   stow_root: /var/lib

- mysql:
   # mysql_root_password: secret
   # mysql_root_password_update: no
   mysql_databases:
     - name: wp_database
   mysql_users:
     - name: wp_user
       host: localhost
       password: wp_user_password
       priv: "wp_database.*:ALL"
```

The `freckle` part determines how the folder is handled by `freckelize`. In this case, we make sure that it's owned by the `mysql` user and group. Also, because we want our data to live under `/var/lib/freckles/wordpress/mysql/mysql` we tell `freckelize` to symbolically link that folder to `/var/lib/mysql`. As a sidenote, this works on Debian Stretch, but I could not make it work on Ubuntu for example, as there is an issue with Apparmor I couldn't figure out.

The second part let's us specify the name of the database we want, as well as a database user and it's permissions. This is more or less directly forwarded to the [Ansible role that handles this](https://github.com/geerlingguy/ansible-role-mysql).

### Wordpress

For the Wordpress part of the data we create a second folder under `/var/lib/freckles/wordpress`, which we call `wordpress`. Within this, we create a folder `wp-content`, and an empty file called `wp-config.php`. We create those now, so that the `stow`'-phase of `freckelize` can symbolically link them even before they are created (by unpacking the wordpress archive after it's downloaded). Additionally, we also create a `.freckle` folder here:

```
- freckle:
   owner: www-data
   group: www-data
   staging_method: stow
   stow_root: /var/www/wordpress

- wordpress:
   wordpress_domain: 127.0.0.1
   wordpress_port: 80
   wordpress_db_name: wp_database
   wordpress_db_host: localhost
   wordpress_db_user: wp_user
   wordpress_db_user_password: wp_user_password
   wordpress_db_table_prefix: wp_
   wordpress_language: en_US
   wordpress_debug: false
   letsencrypt_email: none
```

As before, we set user/group permissions. This time to the user who will run the webserver. We `stow` the folder containing the `.freckle` file as before.

The `wordpress` part specifies some properties related to the database we've setup before, the webserver that will serve our Wordpress site, as well as some Wordpress configuration options to apply if no configuration exists yet. The last property 'letsencrypt_email` works the same way as it is described in the post about the `static-website` adapter, and we'll leave that disabled for now.

That is all the preparation we have to do. Now we can run `freckelize` against the base folder that contains our two freckle folders:

```
freckelize -f /var/lib/freckles/wordpress
```

This will link our empty files and folders into place, then setup MySQL, nginx and Wordpress so we end up with a fresh (and) empty Wordpress instance that we can visit at [http://127.0.0.1](http://127.0.0.1).
