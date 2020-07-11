---
layout: post
title: Downloading Raspberry Pi Press issues
subtitle: And staying up to date on Pi matters
date: "2020-07-09T00:00:00Z"

reading_time: true  # Show estimated reading time?
share: true  # Show social sharing links?
profile: false  # Show author profile?
comments: false  # Show comments?
tags: [tutorial, Raspberry Pi, press, free magazines]
---

In this post we will discover the great magazines edited under the
Raspberry Pi Press umbrella and discuss how to easily access them all.

---
# Content
*   [The Raspberry Pi Press magazines](#the-raspberry-pi-press-magazines)
*   [Bookshelf](#bookshelf)
*   [rpipress-downloader](#rpipress-downloader)
---

# The Raspberry Pi Press magazines

The [Raspberry Pi Press][pi-press-web] is a part of the Raspberry Pi Foundation
and the publisher of a great deal of magazines and books.
Among the many magazines edited, some are freely available for download,
- [HackSpace][hackspace] is a monthly publication dedicated to those who love to make things
and learn while doing it.
- [HelloWorld][helloworld] is published three times a year and targets educators
of the computing and digital world.
- [MagPi][magpi] is the official magazine of the Raspberry Pi and is loaded
with stories and project based on the single board computer.
Published every month, the latest issue is numbered N°95 as of the time of writing,
making it an incredible source of inspiration.
- [Wiredframe][wireframe] is published every 2 weeks and is entirely dedicated
to video games. But unlike other video game magazines, it offers
to look at how they are made, who make them and offer a lot of resources
to get started writing your own games.

On top of that, The Raspberry Pi Press also publishes many great books.

Each magazine can be bought online and shipped around the globe.
One can also sign for a yearly subscription, offering some
discount and/or goodies. At the same time, issues are freely available
to download in pdf from the magazine websites.

# Bookshelf

Recently the [Raspberry Pi Foundation presented the Raspberry Pi OS][pi-os-blog],
a rebranding of Raspbian, highlighting some of its novelties.
Among those novelties, they showcased a neat little app named `Bookshelf`
that allows you to browse and download the issues of
several magazines edited by the Raspberry Pi Press.

The application is a simple interface listing all issues of each magazine
but also some of the books.
It allows for simply downloading any issue by simply clicking on
the desired cover.

![The Bookshelf app](/img/post/bookshelf.png)

Unfortunately this great app is only available through the Pi OS
archive and its [source code is not public][bookshelf-post] at the time of writing.
One can still download the deb package and install it manually.
To do so, visit the [app archive][bookshelf-repo] and look for the latest
version of the debian package for your machine architecture.
At the moment it is `rp-bookshelf_0.4_amd64.deb` for common computers.
From there, we can simply download the debian and install it,
```bash
$ wget http://archive.raspberrypi.org/debian/pool/main/r/rp-bookshelf/rp-bookshelf_0.4_amd64.deb
$ dpkg -i rp-bookshelf_0.4_amd64.deb
```
To launch the app simply type,
```bash
$ rp-bookshelf
```
Altho this procedure works fine, it is a little unpleasant.
Furthermore, I personally don't care much about the GUI and I'd rather prefer
to automatically download the latest issues I care for.
If you feel the same, keep on reading.

# rpipress-downloader

The Raspberry Pi Press Store was [recently entirely redesigned][pi-press-blog]
bringing some uniformization across all the magazine websites.
That allows us to write a small web scrapping script to automatically
download the latest (or all) issues and books of our favorite magazine(s).

So I went ahead and did just that, writing a small Python script that you can find
on [Github][rpipress-github], or conveniently install as a Snap as follows,
```bash
sudo snap install rpipress-downloader
```

Its use it pretty simple, launch the script in a terminal
and by default it will automatically search and download the latest issue of all
aforementioned magazines.

Further options let you:
- specify which magazine to download
  ```bash
  rpipress-downloader --magazines magpi hackspace
  ```
- download **all** issues,
  ```bash
  rpipress-downloader --all
  ```
- download the books too,
  ```bash
  rpipress-downloader --books
  ```
- combine options so that,
  ```bash
  rpipress-downloader -a -m magpi -b
  ```
  will download all MagPi issues and books.

Issues and books are saved respectively in

-  `~/rpipress/{magazine}`
-  `~/rpipress/{magazine}/Books`

or, using the snap, in

-  `~/snap/rpipress-downloader/current/rpipress/{magazine}`
-  `~/snap/rpipress-downloader/current/rpipress/{magazine}/Books`.

Note that the script conveniently let you know the path by printing an
hyperlink in the console,
```bash
$ rpipress-downloader -m magpi
Latest MagPi issue is N°95
You are up to date
Your favorite magazines are waiting for you in file:///home/artivis/snap/rpipress-downloader/5/rpipress
```

Please refer to the [rpipress-downloader][rpipress-github] readme page
for further information.

Have a good reading!


[//]: # (URLs)

[pi-press-web]: https://store.rpipress.cc/
[pi-os-blog]: https://www.raspberrypi.org/blog/latest-raspberry-pi-os-update-may-2020/
[pi-press-blog]: https://www.raspberrypi.org/blog/the-raspberry-pi-press-store-is-looking-mighty-fine/
[rpipress-github]: https://github.com/artivis/rpipress-downloader
[rpipress-snap]: https://snapcraft.io/rpipress-downloader

[hackspace]: https://hackspace.raspberrypi.org/
[helloworld]: https://helloworld.raspberrypi.org/
[magpi]: https://magpi.raspberrypi.org/
[wireframe]: https://wireframe.raspberrypi.org/

[bookshelf-post]: https://www.raspberrypi.org/forums/viewtopic.php?f=63&t=278584&p=1687369&hilit=bookshelf#p1687369
[bookshelf-repo]: http://archive.raspberrypi.org/debian/pool/main/r/rp-bookshelf/
