---
title: "Setting up Fastmail as a desktop email client in Linux Mint"
date: 2025-07-05T00:00:00+02:00
author: "Víctor"
url: "/en/fastmail-desktop-app-linux-mint"
toc: false
draft: true
---

Although the ongoing demise of desktop applications in favour of web apps is affecting all operating systems, Linux users are especially familiar with it. Due to its comparatively small market share, Linux is typically the lowest hanging fruit when it comes to cutting on development efforts. The rise of powerful, highly functional web apps has only accelerated this trend.

Unfortunately, some services and applications can lose valuable functionality when native apps are removed from the equation, and email is a good example of this: background mailbox monitoring, proper notifications, handling of mailto links or offline support are all things that can be lost without a desktop app. Third party email clients (such as Thunderbird) can cover these gaps, but they come with their own set of compromises: loss of provider-specific features, varying levels of quality and UI polish, questionable business models, etc.

Luckily, due to its open source and customizable nature, Linux provides the necessary tools to turn any webmail service, such as Fastmail, into a fully fledged desktop app.

In this post, I'm going to show the steps you can follow to integrate Fastmail as the default email client on Linux Mint 22.1 (Cinnamon Desktop) as tightly as a native desktop app would be. Once done, this is what we will have:

* A standalone email client app, independent from our usual web browser.
* This web app will be registered and recognized as our default email client by the system.
* `mailto:` links will correctly open in this app when clicked from a normal web browser window.
* When a link is clicked inside an email, it will open in a separate, normal web browser window, not inside the web app.
* A separate background process, independent from the web app, will check for new emails and show notifications. Clicking on these notifications will launch the web app.
* A tray icon will show the unread email count and provide a quick way to launch the web app too.
* (Fastmail specific) We will have full offline access to our mailbox, even without an internet connection.

I'm going to focus on Firefox and Fastmail because those are the ones I use, but all of this should be largely applicable to Chrome/Chromium browsers and most web-based email services.

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

That's fine for any email links found within our inbox, but... what about email links we stumble upon during normal web browsing in our main Firefox instance? To address that, we can borrow a little trick from [this reddit user](https://www.reddit.com/r/linuxmint/comments/x9q49o/comment/inqw9qy/):
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

`mailto:` links in Firefox will now be forwarded to our script, which will in turn forward them to our Fastmail web app. Fastmail will take it from there and will correctly parse any variables present in the link, such as the sender address or the subject line.

You can make a quick test by simply typing `mailto:test@test.com` in your Firefox address bar, then pressing enter.

### 3. Opening external links in our normal web browser (Firefox only)

If you're using Firefox, by now you may have noticed an little quirk with this setup.

When clicking on an external link inside any of your emails, the link is not launched in your regular Firefox browser. Instead, it simply opens a new tab *within the Firefox instance of the web app*. This is obviously not what we want, so we're going to fix it.

Luckily, the work has already been done for us by the excellent [PWA Links](https://addons.mozilla.org/es-ES/firefox/addon/pwa-links/) add-on, so all we have to do is follow the [instructions in their GitHub repository](https://github.com/Onred/pwalinks) to install both the add-on and the companion app.

Just make sure to install the add-on in the Firefox instance where your web app lives, NOT on your main Firefox instance.