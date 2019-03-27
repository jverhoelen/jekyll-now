---
layout: post
title: "Introducing a Graphical User Interface for Gopass"
category: "Security"
---

[Gopass](https://github.com/gopasspw/gopass) is "the slightly more awesome standard unix password manager for teams". It combines the powers of Git and GPG to achieve this. Therefor it's purely command line based – and that's great, don't get me wrong. However, non-technie-people within cross-functional teams also need access to a bunch of credentials. For this target group and all the GUI-lovers out there, we are building Gopass UI at codecentric.

If you are curious and lack a detailled introduction into Gopass, feel free to checkout [my blog article about Gopass](https://blog.codecentric.de/en/2019/02/manage-team-passwords-gopass/) on the magnificent codecentric blog.

## Welcome to Gopass UI 

My colleague [Matthias Rütten](https://github.com/ruettenm) and me started to face the challenge to build a simple UI for Gopass when working together in a large cross-functional team. Command line gurus usually fall in love with Gopass. Not so CLI-oriented people on the other side have a hard time setting it up, configuring it and using it. But if you're not familiar with command lines at all, teams still need UIs. This was our motivation behind [Gopass UI](https://github.com/codecentric/gopass-ui).

<img class="img-fluid" src="https://github.com/codecentric/gopass-ui/raw/master/docs/gopass-ui-logo.png" alt="Gopass UI logo" style="width: 500px;margin:0 auto;display:block;">

Gopass UI is available for MacOS, Linux (.rpm, .deb, gentoo) and Windows at the time of writing this blog post. It is written in TypeScript and uses Electron, React and Redux. So under the hood it is just a web application living inside a Chromium browser that connects to a Node.js backend for its system calls. The only pre-requisite before using it is to have Gopass itself installed and configured. So get yourself the [latest release and install it](https://github.com/codecentric/gopass-ui/releases/latest). We are happy that the maintainers of Gopass even let us mention Gopass UI [in the "Download a GUI" section of their setup docs](https://github.com/gopasspw/gopass/blob/master/docs/setup.md#download-a-gui).

Please let us know what you think about it. Of course you are also invited to contribute to the project with us.

<img class="img-fluid" src="https://github.com/codecentric/gopass-ui/raw/master/docs/demo-720p.gif" alt="GIF demonstrating core features of Gopass UI" title="Gopass UI demo" style="width: 800px" />
