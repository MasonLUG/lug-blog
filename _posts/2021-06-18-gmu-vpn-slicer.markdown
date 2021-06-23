---
layout: post
title:  "GMU VPN Slicer"
date:   2021-06-18 9:00:00 -0400
author: Zach Osman
---

Tired by connecting to the GMU VPN manually? Check this out.

## Background

A common problem I found was that I'd want to access GMU resources (namely `zeus.vse.gmu.edu`) while not having my internet be slowed down by being connected to the GMU VPN. 

The solution that worked early in my time at GMU was to ssh jump into `zeus` from `mason.gmu.edu` (another student accessable linux machine), as `mason` was accessable from outside the VPN, whereas `zues` was not. 

Somewhat recently `mason` followed `zues` and changed to no longer be publically accessable, and must be accessed from inside the GMU network, breaking the initial fix. 

## Current Solution

The VPN Slicer allows for traffic directed at specific domains to be routed through the VPN, whilst leaving the remaining traffic untouched. 

For the implementation below, we will route the following domains through the VPN- `zeus-1.vse.gmu.edu zeus-2.vse.gmu.edu zeus.vse.gmu.edu mason.gmu.edu`, however you can add to this list as you see fit.



## Implementation

This can be done multiple ways, below is how I did it.

This is very similar to 

#### Dependencies
<!-- --protocol=anyconnect -->

* [OpenConnect](http://www.infradead.org/openconnect/)

Openconnect can likely be installed by your distribution package manager.

* [Vpn-Slice](https://github.com/dlenski/vpn-slice)

Check the README for the install guide. 

#### First use

`openconnect vpn.gmu.edu -s "vpn-slice zeus.vse.gmu.edu"`

Results:

Before VPN Connection:
![Figure 1]({{site.baseurl}}/assets/2021-05-21-gmu-vpn-slicer/fig1.png)

VPN Connection:
![Figure 2]({{site.baseurl}}/assets/2021-05-21-gmu-vpn-slicer/fig2.png)

After VPN Connection:
![Figure 3]({{site.baseurl}}/assets/2021-05-21-gmu-vpn-slicer/fig3.png)

We can see that this last connection can connect to `zeus.vse.gmu.edu` but _cannot_ connect to `mason.gmu.edu`, as we didn't specify that address in the sliced domain list for the VPN. This is the expected behaviour.


#### Easier Startup

Lets make a directory for this.

`mkdir gmu-vpn-slicer`

and

`cd gmu-vpn-slicer`

We put the below file inside, as this will fill in the prompts given to us during the `openconnect` connection process, and will allow us to automate this later.

(Be sure to fill in VPN_USER with your masonlive ID)

`vpn-slicer.sh`:
```
#!/bin/sh

SERVER=vpn.gmu.edu
SLICE_SERVERS='zeus-1.vse.gmu.edu zeus-2.vse.gmu.edu zeus.vse.gmu.edu mason.gmu.edu'
VPN_USER=
TYPE=STUDENT
PASS_FILE=./.pass_file

cat "$PASS_FILE" | openconnect "$SERVER" -u "$VPN_USER" -s "vpn-slice $SLICE_SERVERS" -g "$TYPE" --passwd-on-stdin -v
```

We need to make this file executable, so:

`chmod +x ./vpn-slicer.sh`

Lastly, we'll need to create a password file, this will store your GMU password, so it's critical that we secure this file. 

`.pass_file`:
```
your-vpn-password-here
```

And we'll want to set that files permissions correctly so other users on your machine can't read it.

`chmod 600 ./.pass_file`

And we're done!

From here you can simply run `./vpn-slicer.sh` from inside the `gmu-vpn-slicer` directory and be connected.


#### Le Sunny D

Welcome to the official home of the most refreshing orange drink in existence.

We can create a SystemD service to have this start and run automatically, lets do that.


We can create the following file and place it in `/etc/systemd/system`, and we won't have to start this manually.

`vpn-slicer.service`: 
```
[Unit]
Description=GMU VPN
After=network.target

[Service]
User=root
ExecStart=/bin/bash ./vpn.sh
WorkingDirectory=/home/USER/PATH/TO/gmu-vpn-slicer
Type=exec
Restart=always

[Install]
WantedBy=multi-user.target
```

Next we load, enable, and start the service file with the following

`sudo systemctl daemon-reload`

`sudo systemctl enable vpn-slicer`

`sudo systemctl start vpn-slicer`


And we're done!