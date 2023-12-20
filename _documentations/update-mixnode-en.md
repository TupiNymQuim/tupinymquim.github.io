---
permalink: /update-mixnode-en/
---

# Update your mixnode the right way
> By supermeia from TupiNymQuim

Please do consider supporting us by staking on one of the following mixnodes:
* [TupiNymQuim-nodes](https://tupinymquim.github.io/nodes/)
* [supermeia-node](https://explorer.nymtech.net/network-components/mixnode/927)

## Table of contents
1. [Context](#context)
    1. [Goal](#goal) 
    2. [Why does the routing score drop](#why-does-the-routing-score-drop)
    3. [Solution overview](#solution-overview)
2. [Guide](#guide)
    1. [First time setup](#first-time-setup)
        1. [Clone mixnode](#clone-mixnode)
        2. [Allow alternate ports](#allow-alternate-ports)
        3. [Storing binaries](#storing-binaries)
        4. [Service files](#service-files)
    2. [Updating](#updating)
        1. [Storing the updated binary](#storing-the-updated-binary)
        2. [Init mixnode](#init-mixnode)
        3. [Start and enable service](#start-and-enable-service)
        4. [Update wallet information](#update-wallet-information)
        5. [Shut down outdated service](#shut-down-outdated-service)
3. [Conclusion](#conclusion)

## Context

If you're here just for the step by step on how to update without dropping the
routing score you can skip directly to the [Guide](#guide).


### Goal
Updating the mixnode is a very simple process but it has an undesired side
effect of dropping the routing score of your node for a period of time. The aim
of this guide is to provide you with a **step by step guide** on how to start
updating your mixnode with a method **that allows you to not drop your
precious routing score**.

### Why does the routing score drop
The reason behind the routing score dropping is not confirmed (at least to my
knowledge) but after some experience running mixnodes and observing their
behavior. It has become clear to me that the drop in routing score is directly
correlated to the process running the mixnode being reset. That means that if
the process of your mixnode running is reset for any given reason (server
shutdown, server reboot, manual interruption, etc.) the routing score of your
mixnode will automatically start going down afterwards.

### Solution overview
Q: So how does one update their mixnode without dropping the routing score?

A: Simply don't stop the mixnode process!

In short, what we will do to avoid stopping the mixnode process is:
* Create a clone of our mixnode on the same VPS
    * It could be done with 2 different VPSes as well in a more simple but more
    expensive manner
* We will run both of them simultaneously on different ports while updating
    * The clone will have the new version of the binary while the
original will have the outdated version
* We will update the ports on the wallet to match the ports of the updated
process, updating our mixnode
* After the time interval of 1 hour we will shut down the
original outdated mixnode process 
    * This time interval has not been thoroughly tested but it has never
    resulted in routing score dropping while developing this method

## Guide
> This guide assumes you have a mixnode already installed with systemd automation
setup, if this is not your case you can see the official
[Nym docs](https://nymtech.net/operators/introduction.html) for a guide on
the installation and automation process.

> If this is your first time using this method you will have to follow the steps
on the [First time setup](#first-time-setup) section.

> This guide was built with VPSes running on Ubuntu 20.04.

* **Throughout this guide the original mixnode will be referred to as
`<DEFAULT>` and is assumed to be running on the default ports. The cloned
mixnode running with alternate ports will be referred to as `<ALTERNATE>`.
Everytime those strings appear inside commands you will need to substitute them
according to your setup.**

* **In this guide the alternate ports used are 1337, 1338 and 1339. That choice
has no particular reason and you can choose other ports with no implication on
the effectiveness of the method.**

### First time setup
Before starting to update without your routing score dropping you will need to
do some initial setup.

#### Clone mixnode
To be able to run two mixnodes processes at the same time we will need to clone
our mixnode.

We can do that by running the following command:

```bash
cp -rf $HOME/.nym/mixnodes/<DEFAULT> $HOME/.nym/mixnodes/<ALTERNATE>
```

#### Allow alternate ports
For the `<ALTERNATE>` mixnode to be able to run we will need to run it with
different ports than the `<DEFAULT>` mixnode. And because of that we need to
allow these alternate ports on our firewall.

We open these ports with the following command:
```bash
sudo ufw allow 1337,1338,1339/tcp
```

You can see the open ports by running:
```bash
sudo ufw status
```

#### Storing binaries
Since our mixnodes will be running on different binaries it's a good idea to
create a directory structure to store the binaries in an organized manner.

We can create the directories for our binaries with the following commands:
```bash
mkdir -p $HOME/binaries/<DEFAULT>
mkdir -p $HOME/binaries/<ALTERNATE>
```
With that we have a directory structure that looks like this:
```
└──── $HOME
    └── binaries
        ├── <DEFAULT>
        └── <ALTERNATE>
```

#### Service files

Now that we have a structure for storing the different binaries, we can create
the service file for our `<ALTERNATE>` mixnode. This service file will be very
similar to the one we have for our `<DEFAULT>` mixnode, we will only change the
path on Execstart.

Create the following service file `/etc/systemd/system/<ALTERNATE>.service` and
put this inside it:
> Substitute \<USER\> by your username and \<VERSION\> by the version of the
binary.
```
[Unit]
Description=Nym Mixnode <VERSION>
StartLimitInterval=350
StartLimitBurst=10

[Service]
User=<USER>
LimitNOFILE=65536
ExecStart=/home/<USER>/binaries/<ALTERNATE>/nym-mixnode run --id <ALTERNATE>
KillSignal=SIGINT
Restart=on-failure
RestartSec=30

[Install]
WantedBy=multi-user.target
```

You should also change the `<DEFAULT>` mixnode service file to point to the new
directory as well, but don't reload the daemon just yet.

modify the service file `/etc/systemd/system/<DEFAULT>.service` to look like 
this:
> Substitute \<USER\> by your username and \<VERSION\> by the version of the
binary
```
[Unit]
Description=Nym Mixnode <VERSION>
StartLimitInterval=350
StartLimitBurst=10

[Service]
User=<USER>
LimitNOFILE=65536
ExecStart=/home/<USER>/binaries/<DEFAULT>/nym-mixnode run --id <DEFAULT>
KillSignal=SIGINT
Restart=on-failure
RestartSec=30

[Install]
WantedBy=multi-user.target
```
With that, all of the setup steps are done and we can proceed to update our
node without dropping the routing score.

### Updating
> In this section it will be assumed you will have done the
[setup](#first-time-setup) presented in the previous section.

**In this section 2 options of commands will be shown throughout:**
* `<DEFAULT>` outdated: if your outdated mixnode is the one with default ports
* `<ALTERNATE>` outdated: if your outdated mixnode is the one with alternate
ports

> If this is your first time using this method you will be following the steps
labeled as "`<DEFAULT`> outdated".

#### Storing the updated binary

If you don't know how to get the binaries you can find instructions on the
official [Nym docs](https://nymtech.net/operators/introduction.html).

After you download or build the updated nym-mixnode binary you will need to
move it to the corresponding directory:

> Substitute \<PATH_TO_UPDATED_BINARY\> to the path where you downloaded or
built the updated binary.

For `<DEFAULT>` outdated.
```bash
mv <PATH_TO_UPDATED_BINARY>/nym-mixnode $HOME/binaries/<ALTERNATE>/
```
For `<ALTERNATE>` outdated.
```bash
mv <PATH_TO_UPDATED_BINARY>/nym-mixnode $HOME/binaries/<DEFAULT>/
```

#### Init mixnode

Now you need to init the node with the new binary:

For `<DEFAULT>` outdated.
```bash
$HOME/binaries/<ALTERNATE>/nym-mixnode init --id <ALTERNATE> --host $(curl -4 https://ifconfig.me) --mix-port 1337 --verloc-port 1338 --http-api-port 1339
```
For `<ALTERNATE>` outdated.
```bash
$HOME/binaries/<DEFAULT>/nym-mixnode init --id <DEFAULT> --host $(curl -4 https://ifconfig.me)
```

#### Start and enable service
Now your mixnode is up to date and we need to get it running, we can do so by
enabling and starting it's service:

For `<DEFAULT>` outdated.
```bash
sudo systemctl enable <ALTERNATE>.service
sudo systemctl start <ALTERNATE>.service
```
For `<ALTERNATE>` outdated.
```bash
sudo systemctl enable <DEFAULT>.service
sudo systemctl start <DEFAULT>.service
```

You can check if the mixnode is running correctly with the following command:

For `<DEFAULT>` outdated.
```bash
systemctl status <ALTERNATE>.service
```
For `<ALTERNATE>` outdated.
```bash
systemctl status <DEFAULT>.service
```

#### Update wallet information
Now you need to update your mixnode ports and version on the wallet, so that
the updated mixnode starts receiving packets.

In order to do this you need to open your wallet, click on Bonding and head to
Node Settings: `Bonding -> Node Settings`

On the node settings page you will need to update the fields accordingly:

For `<DEFAULT>` outdated.
```bash
Mix port = 1337
Verloc port = 1338
HTTP port = 1339
Version = new mixnode version
```

For `<ALTERNATE>` outdated.
```bash
Mix port = 1789
Verloc port = 1790
HTTP port = 8000
Version = new mixnode version
```

After this the updated mixnode will be the one receiving the packets

#### Shut down outdated service
**After waiting 1 hour** from when you updated the ports on the wallet you can
shutdown the service that is running the outdated mixnode:
> The time interval is not yet thoroughly tested, but 1 hour should be a safe
interval for your routing score not to drop

For `<DEFAULT>` outdated.
```bash
sudo systemctl disable <DEFAULT>.service
sudo systemctl stop <DEFAULT>.service
```
For `<ALTERNATE>` outdated.
```bash
sudo systemctl disable <ALTERNATE>.service
sudo systemctl stop <ALTERNATE>.service
```

If this was the first time you updated with this method you should reload the
daemon in order to register the changes we made on the `<DEFAULT>` service file.
You can do so by running the following command:
```bash
sudo systemctl daemon-reload
```
With this your mixnode is now running with the latest binary and it's routing
score has stayed intact.

## Conclusion
If you have any contributions you would like to make, feel free to open a pull
request or an issue on the
[github repository](https://github.com/TupiNymQuim/tupinymquim.github.io).

And if this was useful to you please consider supporting us by staking on one
of the following mixnodes:
* [TupiNymQuim-nodes](https://tupinymquim.github.io/nodes/)
* [supermeia-node](https://explorer.nymtech.net/network-components/mixnode/927)
