#### TL note:
##### All of this I found reading through several overlapping sources as I worked it out. Im just posting with future-self in mind who may not want to piece things together later, and to put it all in one place for anyone with the same goal.

## Using other containers in ChromeOS _Terminal_
This is a guide for setting up the default _Terminal_ application offered by ChromeOS' Linux dev environment. We can achieve the same within the crosh shell by entering containers manually, but that removes several hotkeys from us that a more reasonable (usable) terminal offers than the shell on Chrome Browser. 

We're starting off assuming you have already enabled Dev mode and Linux features are turned on.

For our purposes, refer to this diagram to get an idea of where we are in namespaces.

Ctrl+Alt+t to open up _crosh_ and type
`vmc start termina`
to enter the termina VM.

Once in, you can use
`lxc launch *<remote>*:*<image>*`
-`remote` here being the name of the remote image source
-`image` the name of the image we want to pull down from there over the internet

we can see what remote image servers we have on hand with
`lxc remote list`

In this case, I have:
![](https://user-images.githubusercontent.com/54195989/142089061-34b0a99b-ea12-40b0-b9f9-7c373e5650dd.png)

..by default, which is fine because we have a lot of what we need (or would want) with this. I imagine we can add more image servers, but I did not.

We can check our options for image servers by going to its url. We can see that the [`images`](https://us.lxd.images.canonical.com/) image server (confusingly named, though _tru, I guess_) offers many common Linux distributions.
  
We can use `lxc launch *<remote>*:*<distribution>*/*<release>*/*<architecture>* *<container_name>*`
to install images with varying degrees of specificity.

For example:
alpine, release 3.11, for amd64 (64-bit) architechture from [`images`](https://us.lxd.images.canonical.com/)
`lxc launch images:alpine/3.11/amd64 `

kali container (all else current or default) named BIGHAX from [`images`](https://us.lxd.images.canonical.com/)
`lxc launch images:kali BIGHAX`

Ubuntu 21.04 from [`ubuntu`](https://cloud-images.ubuntu.com/releases/) doesn't seem to offer as many things to specify, though does conveniently list both the release # and codename/'animal' adjective as images, meaning both 
`lxc launch ubuntu:*21.04*`
or 
`lxc launch ubuntu:*hirsute*` work!


For our purposes

Refs:
[Chrome Docs](https://chromium.googlesource.com/chromiumos/docs/+/HEAD/containers_and_vms.md)
[Crostini Docs]
(https://chromium.googlesource.com/chromiumos/docs/+/HEAD/crostini_developer_guide.md)
[Ubuntu Docs](https://ubuntu.com/blog/using-lxd-on-your-chromebook)
[post on /r/Crostini](https://www.reddit.com/r/Crostini/comments/fj8ddg/instructions_for_kali_linux_on_crostini/)
