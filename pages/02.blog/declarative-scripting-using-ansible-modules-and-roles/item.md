---
title: "Declarative scripting, using Ansible modules and roles"
published: true
date: '31-10-2017 15:00'
taxonomy:
    category:
        - blog
    tag:
        - freckles
        - frecklecute
        - ansible
author: 'Markus Binsteiner'
toc:
    headinglevel: 3
---

Ever wanted to just quickly run a few Ansible tasks or apply a role from *Ansible Galaxy*, without having to manually setup *Ansible*, or create an inventory, and/or download role(s) from *Ansible Galaxy*, etc...? Or have you ever wished you could write a re-usable command-line script using *Ansible modules* and *roles*? Or, maybe you haven't thought about it before, but now that I mention it...

If, this here blog post is for you.

===

We're going to be using `frecklecute` for this, which is part of the [*freckles*](https://github.com/makkus/freckles) package. To learn more about *freckles* itself: I've written about what it is and what you can do with it [here](/blog/so-i-made-this-thing).[TOC]

## tldr; show me the goods

Here's one such example script (let's name the file `setup_sudo_user`), which sets up a user account (if it doesn't exist yet), and makes sure the user is part of the `wheel` group (which will also be created if necessary). The script also sets up 'passwordless' *sudo* permissions for that `wheel` group:

```
doc:
  short_help: "setup new sudo user"
  help: "Setups a new user with (passwordless) sudo privileges.\n\nInstalls the 'sudo' package if necessary, and creates a group 'wheel' which will be allowed passwordless sudo-access."
  
args:
  user_name:
     help: the name of the user
     is_var: false
     required: yes
  password:
     help: the user password hash (generate with 'mkpasswd -m sha-512')
     is_var: false
     required: yes
     
tasks:
  - group:
      name: wheel
      state: present
  - package:
      name: sudo
      state: present
  - lineinfile:
      dest: /etc/sudoers
      state: present
      regexp: "^%wheel"
      line: "%wheel ALL=(ALL) NOPASSWD: ALL"
      validate: "/usr/sbin/visudo -cf %s"
  - user:
      name: "{{:: user_name ::}}"
      password: "{{:: password ::}}"
      update_password: always
      groups: wheel
      append: yes
```

To use this script, assuming [we already bootstrapped *freckles*](https://docs.freckles.io/en/latest/readme.html#really-quick-start), all we need to do is (as root in this case):

```
# frecklecute setup_sudo_user --help
Usage: frecklecute setup_sudo_user [OPTIONS]

  Setups a new user with (passwordless) sudo privileges.

  Installs the 'sudo' package if necessary, and creates a group 'wheel'
  which will be allowed passwordless sudo-access.

Options:
  --password TEXT   the user password hash (generate with 'mkpasswd -m
                    sha-512')  [required]
  --user_name TEXT  the name of the user  [required]
  --help            Show this message and exit.

  For more information about frecklecute and the freckles project, please
  visit: https://github.com/makkus/freckles

# mkpasswd -m sha-512 hello_password
$6$h5OgOaSfa$rwIgBF1Ds/YKx9200agirpmdjG/8D5ThsM3AG9ozvlwci3DzZrBcqRA6LbOQMRAStQop0MWlDes5atB/E7BR6.

# frecklecute setup_sudo_user --user_name fancy_new_user --password '$6$h5OgOaSfa$rwIgBF1Ds/YKx9200agirpmdjG/8D5ThsM3AG9ozvlwci3DzZrBcqRA6LbOQMRAStQop0MWlDes5atB/E7BR6'

* starting tasks (on 'localhost')...
 * starting custom tasks:
     * group... ok (changed)
     * package... ok (no change)
     * lineinfile... ok (changed)
     * user... ok (changed)
   => ok (changed)
```


## Uuuuuhhhhhh, nice! Tell me more!

