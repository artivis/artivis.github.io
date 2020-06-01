---
layout: post
title: Managing dotfiles
subtitle: Because there is no place like $HOME
date: "2020-06-01T00:00:00Z"

reading_time: true  # Show estimated reading time?
share: true  # Show social sharing links?
profile: false  # Show author profile?
comments: false  # Show comments?
tags: [tutorial, dotfiles, HOME, homeshick, LXC, LXD]
---

In this post, we will see how we can easily manage our personal
configuration files - a.k.a. dotfiles.
Yeah dotfiles, named after there common `~/.my_config` form, you know,
all of those small configuration files lying across our $HOME.

> Because there is no place like $HOME

Because we are spending so much time on our machine, be it for work or for fun
(both at the same time if you are lucky),
we love to tweak our environment to our taste and needs.
Change the UX, create some aliases, use some dark theme and what not,
most if not all of these is saved in some configuration files somewhere.
And since we spent so much time making a home for ourselves,
wouldn't it be great if we could quickly set it up again on a different computer?
Change the house but keep the furniture and decorations?
This is precisely what we are going to set up here.

# Picking dotfiles manager

Looking on the web for a dotfiles manager, you my find many of them -
see a whole [list of them here][dotfiles-list]. Most of them work off
the same principles, being a small set of utils to help manage our dotfiles.
Management includes most importantly versioning, often through git and
the installation of the files to their correct location as they are more than
often expected to be found at a given path.
You may want to give a look at aforementioned list of managers
and pick to one that best answer your needs and expectations.
Note that many are interchangeable.

In this post we settled using [`homeshick`][homeshick].
There are two main reasons to this choice.
Firstly, it is entirely written in bash, making it usable virtually anywhere.
Secondly, it 'installs' dotfiles on our system using symlinks rather than
hard copies. The files thus exist in a single place.
Some other nice features includes, being git-based, being cli-based,
supporting multi dotfiles repos.
It has to be noted tho that the project is not in a really active
development and not very feature rich compared to other solutions.
It is a thin-layer that does the job.

Alright so how do we get started?

# Building our castle

`homeshick` relies around the concept of *castles* which are nothing
more than git repositories.
A castle contains all of our dotfiles which are organized with the same
layout as our home directory.
But before building our castle, we need to install the appropriate tool.
To install `homeshick`, nothing easier, we simply clone its repository
in our home:
```bash
$ git clone https://github.com/andsens/homeshick.git $HOME/.homesick/repos/homeshick
```
And we are done. Now to use it, we only have to source it,
e.g. directly in our `.bashrc`,
```bash
$ echo "source ~/.homesick/repos/homeshick/homeshick.sh" >> ~/.bashrc
```
We can also source its tab completion tool to ease our life,
```bash
$ echo "source ~/.homesick/repos/homeshick/completions/homeshick-completion.bash" >> ~/.bashrc
```

Alright, we are done with the installation,
let us start creating the said castle.

First we create a new local git repo through `homeshick` cli tool,
```bash
$ homeshick generate dotfiles
```
This creates an empty castle named 'dotfiles' in
`~/.homesick/repos/dotfiles/`.
To populate our castle with a dotfile, we make use of the 'track' command:
```bash
$ homeshick track {castle} {dotfile}
```
To track our first file, say e.g. `.bashrc`, we simply issue,
```bash
$ homeshick track dotfiles ~/.bashrc
```
The command copies the file in our castle at
`~/.homesick/repos/dotfiles/home/.bashrc` and replaces the original file
with a symlink to the copy.

Now all we have to do is to commit our change and save our castle online,

To move to our local repository, we enter,
```bash
$ homeshick cd dotfiles
```
and we can now use the usual git commands,
```bash
$ git add .
$ git commit -m 'add .bashrc'
```
Let us save our castle online, e.g. on GitHub,
```bash
$ git remote add origin git@github.com:user/dotfiles.git
$ git push -u origin master
```

We may now repeat this operation for each and every configuration file
we would like to save.
With our castle safely backed up online, we will now see
how we can quickly set up our environment on a new machine.

# Quickly setting up a new machine

Whether you bought a new computer or nuked your old hardware with a
fresh new distro, you will now witness the true power of `homeshick`.

To install our cosy environment on a fresh distro,
all we have to do is,

1. Install `homeshick`  
    ```bash
    $ git clone https://github.com/andsens/homeshick.git $HOME/.homesick/repos/homeshick
    $ source ~/.homesick/repos/homeshick/homeshick.sh
    ```
2. Import our castle
    ```bash
    $ homeshick clone git@github.com:user/dotfiles.git
    ```
3. Let `homeshick` works its magic
    ```bash
    $ homeshick link dotfiles
    ```
Voila! Home sweet home.

Of course this post is only a quick overview of a given dotfiles manager.
I won't detail here all of its options and features
and let discover them for yourself in its [wiki][homeshick-wiki].
As mentioned previously many dotfiles managers rely on a git repository and the
same layout as `homeshick` so you can get started with it and later move to
another one which better fits you needs.

At this point you may be wondering if this it really worth it given that you probably
install a fresh distro every 2 years or so and completely change hardware even less
frequently.
Well, fellow developer, aren't you using containers?
If not, you definitely should consider it and check [this other post][LXD-post]
were I detail development workflow for [ROS][ROS] in [LXD][LXC].

# Disposable tiny home

If you are like me, trying your best to keep a tidy laptop while
messing around with plenty of different software toys,
then you may have had one of this day during which you spawn several containers.
Containers in which we don't have our sweet bash aliases;
on our very own machine!
But thanks to `homeshick` we can now start up a fresh
container and have it mimic `$HOME` in a matter of seconds!
Let me demonstrate it for you with a LXD container,

```bash
$ lxc launch ubuntu:20.04 tmp-20-04
$ lxc profile add castle tmp-20-04
$ lxc ubuntu tmp-20-04
```
Ahhh, what a cozy tiny disposable home!

That seemed to easy to you? Alright I confess, I used some of my own aliases here.
But isn't it what this whole post is about?
Note that the above 3 lines really boils down to,
```bash
$ lxc launch ubuntu:20.04 tmp-20-04
$ lxc exec tmp-20-04 -- sudo --login --user ubuntu
...
$ git clone https://github.com/andsens/homeshick.git $HOME/.homesick/repos/homeshick
$ source ~/.homesick/repos/homeshick/homeshick.sh
$ homeshick clone git@github.com:user/dotfiles.git
$ homeshick link dotfiles
```

With this example,
I hope that I managed to offer you a glimpse at the power of `homeshick`
(and more generally of dotfiles managers),
especially when coupled to a containerized workflow.

Before closing this post, let me give you one last tip.
Because we made our containerized workflow rather seamless with our
host, it can be easy to loose track of which shell is in a container and which
is not. To differentiate them, add the following to your `.bashrc`:
```bash
function prompt_lxc_header()
{
  if [ -e /dev/lxd/sock ]; then
    echo "[LXC] ";
  fi
}

PS1='$(prompt_lxc_header)'$PS1
```
When used in a container,
a shell prompt in the said container will now look something like:
```bash
[LXC] ubuntu@tmp-20-04:~$
```
No more confusion :+1:

[//]: # (URLs)

[dotfiles-list]: https://dotfiles.github.io/utilities/
[homeshick]: https://github.com/andsens/homeshick
[homesick]: https://github.com/technicalpickles/homesick
[homeshick-wiki]: https://github.com/andsens/homeshick/wiki

[LXD-post]: /post/2020/lxc
[ROS]: https://www.ros.org/
[LXC]: https://linuxcontainers.org/
