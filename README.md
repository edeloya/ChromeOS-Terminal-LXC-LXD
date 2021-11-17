**TL note:**
###### All of this I found reading through several sources with a lot of overlapping information as I worked it out. Im just posting with future-self in mind who may not want to piece things together later, and to put it all in one place for anyone with the same goal.

## Using other containers in ChromeOS (crostini) _Terminal_
This is a guide for setting up the default _Terminal_ app offered by ChromeOS' crostini Linux env. The goal here is to use the _Terminal_ with other Linux distros you may have installed alongside the default **penguin** (stripped-down Debian) container.

We can achieve the same within the _crosh_ shell by just entering containers manually, but that removes access to several default hotkeys/behavior a slightly more hospitable terminal offers over a shell within Chrome. Also its annoying to do every time.

For our purposes, refer to this diagram to get an idea of where we are in namespaces.
![namespaces](LOL come back to this)


#  Guide
### crosh
We're starting off assuming you have already enabled Dev mode and Linux features are turned on.

Ctrl+Alt+t to open up _crosh_.<br>`vmc` at this level manages your VMs. Your shell should start with **crosh>** , letting you know we're at the _topmost_ layer in our namespaces right now:

![crosh](https://user-images.githubusercontent.com/54195989/142129994-b0bee437-969e-47a7-b76f-aa5161fbf870.png)

You can use <code>vmc destroy **vm_name**</code> and <code>vmc start **vm_name**</code> to quickly burn down or spin up new VMs. Use `vmc list` to check what VMs exist. `create` can also be used instead of `start`, the difference being that `start` fully initiates a VM and also enters it. 

type
<code>vmc start **termina**</code>
to enter the default _`termina`_ VM.

![termina](https://user-images.githubusercontent.com/54195989/142133934-6abde3ba-3eb8-4434-9fe1-05427674a725.png)

Once in the _`termina`_ layer, you can use:

<code>lxc launch **_remote_**:**image**</code>
<br>•**_`remote`_** here being the name of the remote image source
<br>•**`image`** the name of the image we want to pull down over the internet

we can see what remote image servers we have on hand with `lxc remote list`

In this case, I have:
![remote_list](https://user-images.githubusercontent.com/54195989/142089061-34b0a99b-ea12-40b0-b9f9-7c373e5650dd.png)
by default, which is fine because we can access what we need (or would want?) with this. I imagine we can add more image servers, but I did not.

We can check an image server's wares by going to its url. We can see that the [`images`](https://us.lxd.images.canonical.com/) image server (confusingly named, though _tru, I guess_) offers many common Linux distributions.
  
We can use:
<br><code>lxc launch _**images**_:**distribution[/release][/architecture]** **container_name**</code>
<br>to install images with varying degrees of specificity from there (values in _**[ ]**_ are optional).

<br>For example:
<br>centos, release 7, for amd64 (64-bit) architechture from [`images`](https://us.lxd.images.canonical.com/) with its default name
<br><code>lxc launch _**images**_:**centos/7/amd64**</code>

alpine, from its development branch _edge_ image, named alp-edge
<br><code>lxc launch _**images**_:**alpine/edge alph-edge**</code>

kali container (all else default/current)
<br><code>lxc launch _**images**_:**kali**</code>

<br>[`ubuntu`](https://cloud-images.ubuntu.com/releases/) doesn't seem to offer as many things to specify, though does list both the release # and codename/'animal' adjective, meaning both <code>lxc launch _**ubuntu**_:**21.04**</code> and <code>lxc launch _**ubuntu**_:**hirsute**</code> work.

For our purposes, we're temporarily spinning up an **Ubuntu 18.04** image in specific to steal its set-up LXD client since that was the quickest way to get this done. I believe you can just install an LXD client manually inside the `penguin` container, though I used this method for its ease because of my unfamiliarity with LXD/LXC.

Do that with:
<br>`lxc launch ubuntu:18.04`

![list](https://user-images.githubusercontent.com/54195989/142137304-1665d3e8-8bc4-4df7-897a-6a5a0a394598.png)
<br>If we <code>lxc **list**</code>, we see the current containers. Then we can just `pull` what we need from that container to a folder in /tmp/, and `push` it to the *penguin* container with:

<code>lxc file pull **container_name**/usr/bin/lxc /tmp/lxc</code><br>
<code>lxc file push /tmp/lxc penguin/usr/local/bin/</code>

Then, still within _`termina`_, we enable some settings for LXD:
<br><code>lxc config set core.https_address **:8443**</code>
<br><code>lxc config set core.trust_password **some_password**</code>

We can check they're applied with `lxc config show`.
<br>**Feel free to delete the Ubuntu 18.04 container once done copying LXC over**
<br><code>lxc delete **container_name**</code>

### Terminal
Now we can return to the _Terminal_ to hook it up to LXD, giving us access to our containers from within **penguin**.

Start by getting the container's gateway with `ip -4 route show`, in this case *100.115.92.193*
![ip](https://user-images.githubusercontent.com/54195989/142144234-4a1a3d72-d3b2-408b-a331-0ad42c30035e.png)

This is the internal ip that connects the _`termina`_ *chronos* user to its containers. We're going to connect so we can communicate with LXC within _Terminal_.
<br><code>lxc remote add **chronos** **ip_address** </code>

<br>then we set *chronos* as the default.
<br><code>lxc remote set-default chronos</code>
<br>![lxc_remote](https://user-images.githubusercontent.com/54195989/142146070-51bdea29-69e1-4fdf-820c-707f0ab95dc9.png)

Now within the _Terminal_ app / **penguin** container, you will send LXC commands 'up' a layer to be ran by _`termina`_. You can enter any container you launch with LXC using the command <br><code>lxc exec **container_name** -- bash</code> though I recommend aliasing to just the container name for convenience.

<br><br>Refs:
<br>[Chrome Docs](https://chromium.googlesource.com/chromiumos/docs/+/HEAD/containers_and_vms.md)
<br>[Crostini Docs](https://chromium.googlesource.com/chromiumos/docs/+/HEAD/crostini_developer_guide.md)
<br>[Ubuntu Docs](https://ubuntu.com/blog/using-lxd-on-your-chromebook)
<br>[post on /r/Crostini](https://www.reddit.com/r/Crostini/comments/fj8ddg/instructions_for_kali_linux_on_crostini/)
<br>Every `help` command
