---
layout: post
title: My website
subtitle: Or how to get yours
date: "2020-05-09T00:00:00Z"

reading_time: true  # Show estimated reading time?
share: true  # Show social sharing links?
profile: false  # Show author profile?
comments: false  # Show comments?
tags: [tutorial, website, github, hugo, academic]
---

Here we are, looking for online visibility.
How does one set that up quickly when starting from scratch?
Do **you** Remember those HTML courses?  
**Yeah me neither.**  
But fortunately for us [GitHub](https://github.com/) made it easier than ever!

We will discuss in this post how to set up the necessary tools to build and view and website locally and then how to deploy it to `GitHub`.
The prerequisites are,
* GitHub
* markdown
* LXD (optional)

You may find `GitHub` tutorials [here](https://guides.github.com/activities/hello-world/)
and [there](https://opensource.com/article/18/1/step-step-guide-git).
Pages of our website will be written in [markdown](https://en.wikipedia.org/wiki/Markdown).
You can learn more about markdown from those tutorials,
[in English](http://markdowntutorial.com/) and [in French](https://openclassrooms.com/courses/redigez-en-markdown).
And if you only need a brief refresh,
[here](https://sourcethemes.com/academic/docs/writing-markdown-latex/) is the syntax supported by our website.
Finally, you can find a `LXD` tutorial [here](https://linuxcontainers.org/lxd/getting-started-cli/).

Now, let us set up the necessary stuff to get started, shall we?

# Picking a website template

To build our website, we will use the framework [Hugo](https://gohugo.io/).
It is very convenient for our use case because it comes with a ton of
[predefined website themes](https://themes.gohugo.io/)
and it is very simple to use.

For the purpose of this tutorial we will use the very theme of this website,
namely, [Academic](https://themes.gohugo.io/academic/).
This theme is rather clean, well organized and quite simple too.
Unlike other themes that require some extra steps,
`Academic` is pretty much clone and play.
Furthermore, it is [well documented](https://sourcethemes.com/academic/docs/)!
However, at the time of writing, this theme requires `Hugo *Extended*` version 0.67+.
This distinction is important because,
while it is conveniently [packaged as a snap](https://snapcraft.io/hugo),
the snap only offers the classic version, not the *Extended*.
Therefore we have to fetch its debian package and install it manually.

First, let us clone the `Academic` theme on our machine:
```bash
mkdir ~/workspace
cd ~/workspace
git clone https://github.com/sourcethemes/academic-kickstart.git my_website
cd my_website
git submodule update --init --recursive
```

# Prepping the tools

To avoid polluting our system, we will set up a Linux container in which we will
install `Hugo Extended`.
The container is totally optional and you can do the installation directly on your machine.
If you do not wish to use a container, skip directly to [Hugo installation](#installing-hugo)

## Setting up the LXC

Let us start a fresh and pull a new `Ubuntu 18.04` instance,
```bash
lxc launch ubuntu:18.04 hugo
```

We will now mount a disk device to share the website source code between our machine and the container:
```bash
lxc config device add hugo workspace disk source=~/workspace/my_website path=/home/ubuntu/my_website
lxc config set hugo raw.idmap "both $(id -u) $(id -g)"
lxc restart hugo
```

The default installation of `LXD` set up a bridged network so that containers
live behind a NAT on the host. Therefore, we have to forward the port on which
our website is served by the `Hugo` framework.
To do so, issue the following command:
```bash
lxc config device add hugo proxy1313 proxy connect=tcp:127.0.0.1:1313 listen=tcp:0.0.0.0:1313
```

The container is all set up. We can log to it with:
```bash
lxc exec hugo -- su --login ubuntu
```

## Installing Hugo

We will download the `Hugo` extended debian directly from it `GitHub` repository.
To do so, enter in the terminal:
```bash
wget https://github.com/gohugoio/hugo/releases/download/v0.70.0/hugo_extended_0.70.0_Linux-64bit.deb
```
At the time of writing, the latest `Hugo Extended` release is version 0.70.0.

We can now install it with:
```bash
sudo dpkg -i hugo_extended_*.deb
```

## First view of our website

Let the show begin. We are now ready to spawn our website and browse it.
In a terminal, enter:
```bash
cd /home/ubuntu/my_website
hugo server
```
Voila!

The website it up and running! To visualize it,
open your web browser at the address `http://localhost:1313`.
That was easy right?

# Making the website your own

We have a great template up and running, it is now time to make it our own.
The `Academic` theme comes with a ton of options and configurations allowing us
to truly personalize it to our liking and use case.
And since it online documentation is so great,
I will let you discovers by yourself all the possibilities the theme offers.
Head down to the [Academic get started documentation](https://sourcethemes.com/academic/docs/get-started/)
and have fun!

Just a quick advice, as you edit your website, let `Hugo` run.
It is able to update the website live so that you see your changes take effect
immediately in your web browser!

# Deploying the website to GitHub

Once our website is ready to be made public, all there is to do is to push it to `GitHub`.
In your `GitHub` account, we will create a repository to host your website.
To do so hit the tiny cross `(+)` in the top-right of `GitHub` and select `new repository`.
For `GitHub` to be able to figure out that this particular repository is your personal website
we need to give it a specific name in the form : **<your-github-user-name>.github.io**.

All we have to do now is to push our website to this repository.

First we will commit all of our changes,
```bash
cd /home/ubuntu/my_website
git add .
git commit 'made the website my own'
```

and add the `GitHub` repository we just created as our remote,
```bash
git add remote origin https://github.com/<your-github-user-name>/<your-github-user-name>.github.io.git
```

Let us push the changes upstream,
```bash
git push origin
```
Voila!

Your website is now available at the address  

`https://<your-github-user-name>.github.io/`

Our job here is done.

# Bonus: Academic publications

If you happen to have some academic publications that you would like to showcase
on your website, we will install a Python tool called `academic`
that will help us to automatically generate pages from `Bibtex`.

First we will install `pip3`:
```bash
apt install python3-pip
```

Then we will install `academic`:
```bash
pip3 install -U academic
```

Given that we have a `.bib` file that contains all of our publications,
we can generate the pages as follows:
```bash
cd /home/ubuntu/my_website
academic import --bibtex <path_to_your/publications.bib>
```

You can find more information in the
[Academic theme documentation](https://sourcethemes.com/academic/docs/managing-content/#create-a-publication).
