---
layout: my-post
title: "Trying Out Chrome Remote Desktop"
date: 2025-07-11 00:00:00 +0000
categories: web-browser google-chrome
page_name: set-up-chrome-remote-desktop-en
lang: en
---

This article explains how to use Chrome Remote Desktop to control one computer from another.

## Reference
- [【AV1 VS VP9】AV1とVP9の違い：次世代ビデオコーデック徹底比較（画質、圧縮率、互換性、応用…） - Digiarty WinXDVD](https://www.winxdvd.com/video-convert/av1-vs-vp9.htm)

## Environment
### Destination PC
- Windows 11
- Google Chrome 127.0.6533.89

### Source PC
- Windows 10 64-bit
- Google Chrome 127.0.6533.89

## Table of Contents
- [Set up Chrome Remote Desktop on the destination PC](#set-up-chrome-remote-desktop-on-the-destination-pc)
- [Set up Chrome Remote Desktop on the source PC](#set-up-chrome-remote-desktop-on-the-source-pc)
- [About the options panel](#about-the-options-panel)
- [Remove the destination PC setup](#remove-the-destination-pc-setup)

## Set up Chrome Remote Desktop on the destination PC
Start by setting up Chrome Remote Desktop on the destination PC.

Open Chrome and go to the [Chrome Web Store](https://chromewebstore.google.com/) to add Chrome Remote Desktop to Chrome.  
You’ll need to be signed in to your Google account in Chrome.

![Chrome Remote Desktop page on the Chrome Web Store](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Chrome Remote Desktop page on the Chrome Web Store")

From Chrome extensions, open Chrome Remote Desktop.

![Chrome extensions list](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "Chrome extensions list")

Click the download button for setting up remote access.

![Download button for installer](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image3.png "Download button for installer")

Read the Terms of Service and Privacy Policy, then click “Accept & Install.”

![Ready to install screen](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image4.png "Ready to install screen")

Run the installer named `chromeremotedesktophost.msi` from the status bar or your Downloads folder.

Return to the remote access setup screen and click “Turn On.”

![Allow remote access button](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image5.png "Allow remote access button")

Enter a name for your PC.

![Choose name screen](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image6.png "Choose name screen")

Enter a PIN of your choice.

![Enter PIN screen](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image7.png "Enter PIN screen")

Click “Start” to allow this PC to be accessed from another device.

![List of available PCs to connect](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image8.png "List of available PCs to connect")

## Set up Chrome Remote Desktop on the source PC
Now set up Chrome Remote Desktop on the source PC.

Open Chrome and visit [Chrome Remote Desktop](https://remotedesktop.google.com/access).  
Make sure you're signed in with the same Google account as the one used on the destination PC.

Click the PC you enabled access on.

![List of available PCs to connect](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image9.png "List of available PCs to connect")

Enter the PIN you set up on the destination PC.

![Enter PIN screen](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image10.png "Enter PIN screen")

You can now control the destination PC.

## About the options panel
After connecting to a PC, you can click the “>” symbol on the right side of the screen to open the options panel.
In the options panel, you can disconnect or adjust various connection settings.

![Options panel](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image11.png "Options panel")

Here are some of the available options:

### Disconnecting
Click “Disconnect” at the top left of the options panel to end the remote session.

![Disconnect button](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image12.png "Disconnect button")

### Video Codec
You can choose from three video codecs (compression formats): VP8, VP9, and AV1.
According to this reference page, here’s a rough comparison:

|Item|Order|
|----|----|
|Network bandwidth usage|VP8 > VP9 > AV1|
|Host resource usage|AV1 > VP9 > VP8|

AV1 requires more processing power but offers higher compression, resulting in lower data usage and faster connection speeds.

VP8 requires less processing power but offers lower compression, which may lead to more data usage and slower speeds.

## Remove the destination PC setup
To remove the destination PC setup you configured [here](#set-up-chrome-remote-desktop-on-the-destination-pc), visit [Chrome Remote Desktop](https://remotedesktop.google.com/access) and click the trash can icon.

![Disable remote access button](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image13.png "Disable remote access button")

You can likely perform this action from either the destination PC or the source PC.