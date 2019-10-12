---
title: 'macOS Mojave Dynamic Wallpaper of the Earth'
description: 'Two dynamic wallpapers of our beautiful planet'

draft: false

categories:
  - macOS
  - TUDelft
tags:
  - macOS
  - Dynamic Wallpaper
  - Earth
  - Himawari-8
---

# Change your perspective on the world. Every 90 minutes.

Recently I came upon a Medium post displaying some of the coolest dynamic desktop wallpapers for
macOS. I was amazed: to be honest, I didn't even know this feature existed within macOS outside of
the default wallpapers. One of the featured images in the article was a day-night cycle sattelite
view of the Earth. I decided to try it right away, however I quickly found out that the resolution
of the photos was severely lacking. However, i thought the idea was really cool and beautiful so I
decided to try it for myself.

I spent a couple of days searching for high quality imagery of the Earth and finally found the
website hosting the images captured by the Japanese Himawari-8 sattelite. These images are 100 MP in
size and are uploaded every 10 minutes to an Amazon S3 Bucket.

I decided to compile my dynamic desktop out of 16 photos, so all of the photos are taken at an
interval of 1.5 hours each. Since the photos were about 100 MB each, with the edges of the Earth
touching the frame of the image, I made a Photoshop batch script to automate the creation of a
bigger canvas, applying a
black background and resizing it to 5k resolution. Then, I used an online tool called from
DynamicWallpaperClub to put all of these images into one .HEIC file to finalize the dynamics
wallpaper.

Below, you can see an animation of one full cycle of the wallpaper. It starts at 12 midnight and
also ends there. If you want to download the original wallpaper, then you can do so [here](https://dynamicwallpaper.club/wallpaper/ruoaw3gard).

<video width="100%" height="100%" controls>
    <source src="/static/vid/Earth.mp4">
</video>
