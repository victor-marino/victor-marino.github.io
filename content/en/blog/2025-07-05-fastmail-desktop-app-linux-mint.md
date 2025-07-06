---
title: "Setting up Fastmail as a desktop email client in Linux Mint"
date: 2025-07-05T00:00:00+02:00
author: "Víctor"
url: "/en/fastmail-desktop-app-linux-mint"
toc: false
draft: true
---

Although the ongoing demise of desktop applications in favour of web apps is affecting users of all operating systems, Linux users are especially familiar with it. Due to its comparatively small market share, Linux is typically the lowest hanging fruit when it comes to cutting on development efforts, and the rise of powerful, highly functional web apps has only accelerated this trend.

Luckily, its open-source and highly customizable nature also provide the necessary tools to turn a webmail based service, such as Fastmail, in a fully fledged desktop client.

In this post, I'm going to show the steps you can follow to integrate Fastmail as the default email client on Linux Mint. Once done, we will have:
* A standalone email app separated from your usual web browser.
* External links inside your emails correctly opening in your normal web browser, not the web app.
* App will be registered as default email client at OS level.
* App will become the default handler of `mailto:` links in your web browser.
* A separate background process will check for new emails and showing notifications when they arrive.
* Clicking on email notifications will launch the email client.
* A tray icon will show the unread email count and provide a quick way to launch the app.

Let's get started!

