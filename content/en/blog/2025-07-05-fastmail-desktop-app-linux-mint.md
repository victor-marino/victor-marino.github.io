---
title: "Setting up Fastmail as a desktop email client in Linux Mint"
date: 2025-07-05T00:00:00+02:00
author: "Víctor"
url: "/en/fastmail-desktop-app-linux-mint"
toc: false
draft: true
---

Although the ongoing demise of desktop applications in favour of web apps is affecting all operating systems, Linux users are especially familiar with it. Due to its comparatively small market share, Linux is typically the lowest hanging fruit when it comes to cutting on development efforts. The rise of powerful, highly functional web apps has only accelerated this trend.

Unfortunately, some services and applications can lose valuable functionality when native apps are removed from the equation, and email is a good example of this: background email monitoring, proper notification support, handling of mailto links or offline support are all things that can be lost without a desktop app.

Luckily, Linux's open source and highly customizable nature provide the necessary tools to turn a webmail based service, such as Fastmail, in a fully fledged desktop client with all the bells and whistles.

In this post, I'm going to show the steps you can follow to integrate Fastmail as the default email client on Linux Mint 22.1 (Cinnamon Desktop) as tightly as a native desktop app would be. Once done, this is what we will have:

* A standalone email client app, independent from our usual web browser.
* This web app will be registered and recognized as our default email client by the system.
* `mailto:` links will correctly open in this app when clicked from a normal web browser.
* When a link is clicked inside an email, it will open in the separate, normal web browser, not the web app.
* A separate background process, independent from the web app, will check for new emails and show notifications. Clicking on these notifications will launch the email client.
* A tray icon will show the unread email count and provide a quick way to launch the app too.
* (Fastmail specific) Full offline access to your inbox even if there's no internet connection.

I'm going to focus on Firefox because that's the browser I use, but it should be possible to replicate this with Chrome/Chromium and its derivatives without any issues.

Let's get started!

### 1. Creating the web app

The easy part. Let's launch Mint's built-in "Web Apps" feature and create a new one:

![](./../../assets/fastmail_desktop_app/01.png)

No need to change any defaults. Mint even includes an icon for Fastmail :-)

> Note: I'm using the `betaapp.fastmail.com` domain only because I'm interested in Fastmail's offline mode, which at the time of this writing is a beta feature. You can of course use the regular `app.fastmail.com` domain if you prefer to stick to the stable version.

After doing this, you'll notice there's a new Fastmail app in your start menu. But what did we actually do here?

If you head over to `~/.local/share/applications/`, you'll find a new file called `WebApp-FastmailXXXX.desktop`, where `XXXX` is a random 4 digit number. This is simply a [desktop entry](https://wiki.archlinux.org/title/Desktop_entries) that, when executed, will open the Fastmail website in a dedicated Firefox instance, using a profile that's completely independent from the one used by your regular Firefox browser.

This means you can apply different settings, themes or extensions to this Firefox instance, and those changes will not affect your normal browsing experience. This is very useful for several reasons, as we will see later.

Feel free to pin this new app to your panel or any other place that's convenient to you.


### 2. Opening 'mailto:' links in our new web app

While we're in our new web app, let's open the Fastmail settings, then go to "Mail preferences" and scroll down until we find a button that says `Make the default email app.`:

![](./../../assets/fastmail_desktop_app/02.png)

Clicking this button will ensure any `mailto:` links detected *in this Firefox instance* will be correctly handled by Fastmail.

That's fine for email links found within our inbox, but... how do we make it so that email links in our main Firefox browser are also opened here, and not in a Fastmail tab in that browser? To solve this issue, let's do a little trick:
1. Create a new text file anywhere in your home directory called `fastmail-link-handler.sh`, then paste the following contents in it:
    ```bash
    #!/usr/bin/bash
    XAPP_FORCE_GTKWINDOW_ICON=webapp-manager
    firefox \
        --class WebApp-FastmailXXXX \
        --profile /home/<YOUR_HOME_FOLDER>/.local/share/ice/firefox/FastmailXXXX \
        "$@"
    ```
    Make sure to replace `<YOUR_HOME_FOLDER>` accordingly, and `XXXX` with the real numbers from the name of your `.desktop` file created in step 1.
2. Make the script executable. I placed mine in `~/scripts/`, so I did:
    ```bash
    $ sudo chmod +x ~/scripts/fastmail-link-handler.sh
    ```
3. Launch your *normal* Firefox browser, then open Settings, and scroll down until you find the "Applications" section. Here you can define the default apps for different types of files:

    ![](./../../assets/fastmail_desktop_app/03.png)

    Open the dropdown for the "mailto" type, pick "Use other...", and then browse to your home folder and select the script you just created. The result should be similar to the screenshot above.

That's it!

`mailto:` links in Firefox will be forwarded to our script, which will in turn forward them to our Fastmail web app. Fastmail will take it from there and will correctly parse any variables present in the link, such as the sender address or the subject line.

You can make a quick test by simply typing `mailto:test@test.com` in your Firefox address bar, then pressing enter.