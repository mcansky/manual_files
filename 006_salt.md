# SaltStack

When the number of servers to manage at any single time started to grow past 2 system administrators started to look for ways to automate their actions accross all of them.

The early days were probably made of scripts using ssh keys and connecting to several hosts one after the other to run a single shell script.

Things tend to get complicated fast after that. You can easily figure out that a Mail server and a Database server have different needs. And well, you also need to keep them up to date especially with security patches.

One of the first tools to bring some order was cfEngine. It allowed system administrators to manage a whole park of hosts and each specific need in there.

With time the number of hosts to manage continued to grew and new technologies to communicate with remote servers also appeared.

Programming languages evolved too. As a direct decendent of Perl Ruby was a good candidate to develop tools to manage configuration files. The flexibility of the syntax and the number of libraries available made it a very very good replacement for Perl or C.

Puppet and Chef were written in Ruby and are still very good replacement for cfEngine. Powerful and now well known accross big communities of system admistrators they certainly played a big part in the "Devops" movement.

Yet they were not perfect, as usual choices were made that did not fit some use cases and tastes.

After some time lessons were learned and listed by some people and they decided to start a new generation of configuration management tools that were more adapted to the very diverse and unstable world of the Cloud.

Tools like Ansible and SaltStack are in that last category and give another layer of abstraction to the way one can interact with the infrastructure layer.

## SaltStack

While Chef took me a while to figure out Salt only took me a couple of hours to start with.

Just like Chef or Pupper you usually need a central server to host your configuration and orchestrate other servers and actions on those.

That server is called the *master*, the servers it interacts and orchestrate are called *minions*.

### Salt master

The first thing you want to do is setup a master. You just need to spin an instance (even a small one) on your favorite IaaS and then install salt-master on it.

The next step is to edit the configuration file (/etc/salt/master) to define where it's going to get its configuration files from.

But even before that you can already have minions under its thumb.

Figure out the IP address on which the minions will be talking to it and go with spinning up a new instance in your IaaS and install *salt-minion* on it.

Once that that is done open up the /etc/salt/minion configuration file to edit the ip address of the master. Restart the salt0minion daemon and go back to the master.

### Keys

Not all hosts can access your master. To avoid such troubles you have to validate the minions the first time they try to connect.

```
salt-key -L
```

Will list the currently authorized, banned and not yet authorized keys.

In your case you should see the minion listed in the "not yet authorized" section.

```
salt-key -A
```

Will let you authorize all pending keys in one go.

Once that is done you can actually talk to the minion from the master.

### Are you there ?

You can quickly check if the minion is responding by using the ping command :

```
salt * test.ping
```

This will return you the result of the ping for each host (since we used a *).

### Commands

One of the original use of salt was to execute commands on a whole set of minions in one go.

To do so you can use the *cmd.run* command :

```
salt * cmd.run "w"
```

§Yep, you can basically do what ever you want from there. You are root after all.

### What is the minion made of ?

One big concept for later is the one of grains. 

Grains are bits of information related to a host hardware and software. You can find things like the architecture of the processor, the operating system name (linux, darwin, ...), the RAM quantity, the ip addresses etc ...

You can list those from the master by using the *salt* command :

```
salt * grains.items
```

This will list all grains on all minions (hence the use of *).

If you wanted to only see one minion grains you could have used the "-C" option to do a more clever match :

```
salt -C "host1*" grains.items
```

If it's too much to see in one go you can even check only one specific grain :

```
salt -C "host1*" grains.item system
```

That one will only display the items included in the system grain.

Those are not very helpful for now but will come in handy when you will start to use Salt to define how machines need to be configured.

### States

To do so Salt has a concept of states.

Those files defining actions to execute on minions are called "states". Think of them as defining states into which you want to put minions.

Those states *must not* contain confidential infomration, only generic configuration files or templates to generate them.

They are the basic bricks that allow you to install packages and execute commands on hosts.

#### Distributing states

As you don't put confidential informations in the states you usually put them on a public git (or hg) repository. You then define that repository in your /etc/salt/master. Every 60 seconds Salt will fetch any updates from the repository.

#### Targets

Everything starts with the *top.sls* file at the root of your states directory. There you can start targeting states to specific hosts.

```
  host1*:
    - vim
```

This tells salt that it should apply the state named "vim" for the hosts whose hostname starts with "host1".

Now that "vim" state can be defined in a file named "vim.sls" at the root of the states directory or as a "init.sls" file in a directory named "vim".

#### Installing a package

Let's say you want to install the vim package in that state :

```
vim:
  pkg.install:
    name: 'vim'
```

Pretty simple.

You can do even more like run commands etc ..

#### Alter state on grains

The grains can be used in a state to alter the result depending on an item value.

To do that you can use the following syntax :

```
vim:
  pkg.install:
    {% if grains['distrib'] == 'Ubuntu' %}
    name: vim
    {% else %}
    name: vim-core
    {% end %}
```

Notice that the lines for the content are indented just like they would if the jinja syntax was not there. It's important.

> Notice that grains items names and package names are incorrect here.

#### Pillars

Since grains are inherited from the minion operating system they are not enough to target specific states.

One thing you can do is populate a grains file on the machine at bootstrap time with your custom grains for each machine. It's handy and helpful but not always practical.

Another way to put a bit more meaning in your states and especially private information for your configuration file templates for example is to use pillars.

Pillars are yet another set of files that can be targeted and included for hosts. That data can be confidential information and so this set of files must be hosted in a private repository of sort.

You need to configure that in the master configuration file too.

Usually they are used to include things such as ssh public keys, usernames, hostnames, ips of specific services etc ...

In a very similar manner to the grains you can access the pillar data in console and in the states and templates.

In console you can use the following :

```
# salt * pillar.items
```

And you can also just inspect one specific item :

```
# salt * pillar.item ssh_keys
```

In a state or template you can use the following :

```
{% if pillar['ssh_keys'] %}
```

#### Getting a value or using a default

With both grains and pillars you should always have a fallback value to use in your conditionnals or templates.

The way to do that is by using the 'get' command :

```
{{ salt['pillar.get']('system:blob', 'default') }}
```

You can do something similar with grains :

{{ salt['grains.get']('system:blob'. 'default') }}

It's worth noting that there are several other commands such as those available. Just look a bit in the grains and pillars docs.


### Salt Cloud

Salt cloud is a recent addition that gives you the ability to define templates for servers to create with IaaS providers.

You need to configure templates and then you are able to create instances on the fly.

This is a bit complex and it's better to look into the documentation at this point.


## Epilogue

Salt and ansible are great tools that makes it easy to manage 1+ hosts, maintain them up to date, run a quick command on a whole fleet, etc ...

Fixing a security problem can be a matter of few minutes even with 50+ hosts to work with.

There is another set of features that are to be seen with Salt : running a set of commands (think deploy scenario) on a series of hosts. That one can replace things such as capistrano for example.

