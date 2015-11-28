---
layout: post
title: Ripping tilesets with Imagemagick
date: 2014-12-10 22:54:27.000000000 -05:00
---
As a side project, I've been writing a small, FFIV-style RPG. I choose this style because for it's charm and, I thought, simplicity. However, as I work on it more and more, I've come to realize just how many peices there are to a fully functioning game.

One piece that's been a real limitation has been my lack of artwork. Artwork is essential to game development. It guides your work, flushes out your vision, and make you excited to write more.

However, I'm a crappy artist. What I really need some good provisional artwork similar in style to what I ultimately want.

>Good artists copy; great artists steal.
-Picasso

Lucky me, lots of great game resources are out there that have been ripped from comercial games. As long as I'm not releasing my game with them, I don't feel any guilt in using them. There are [lots](http://www.spriters-resource.com/) [of](http://opengameart.org/) [resource](http://spritedatabase.net/) [sites](http://www.videogamesprites.net/), but I what I want right now is a village tileset from [Final Fantasy Adventure](http://en.wikipedia.org/wiki/Final_Fantasy_Adventure).

I manage to find this screenshot.

![original_image](/content/images/2014/Dec/original.png)

These are the tiles I want! All I need is to abstract them into a tileset that my map editor can load. Lucky me, I have an trick up my sleeve, [ImageMagick](http://www.imagemagick.org/).

First things first, I need to isolate the screenshot and crop it to its correct size. In my handy point-and-grunt system image preview tool, I pull out the 320x256 px screenshot such that it lines up perfectly with the 16x16 px tiles.

![cropped_image](/content/images/2014/Dec/cropped_image.png)

Next, I normalize the image. My strategy requires that, in memory, every tree looks __exactly__ like every other tree. Even though the trees were originally rendered identically (probably by an emulator), due to image format's compression, the exact color of the pixles don't necessarily match anymore.

Luckily, the GameBoy only supports 4 colors (white, light gray, gray, dark gray). All I need to do is use ImageMagick's convert the image for a .bmp with a depth of 4 and the tool with match each pixle back to it's original color.

```
$> convert cropped_image.png -depth 4 cropped_image.bmp
```

Next, I split that image into 16x16 px tiles. For this, I use ImageMagick's convert command, again, and stuff them into a ```tiles/``` directory with incremementing numbers.

```
$> convert -crop 16x16 cropped_image.bmp tiles/tile_%d.bmp
```

Now ```/tiles``` is filled with 16x16 tiles looking like this.

![tiles_0](/content/images/2014/Dec/tile_0.png)

But, there are 320 of them! We need to dedupe! Good thing we normalized. If we inspect the actual files of two matching sprites, they're actually identical!

```
# tree and grass tiles, difference
$> diff tile_0.bmp tile_30.bmp
Binary files tile_0.bmp and tile_30.bmp differ

# tree and tree, no difference
$> diff tile_0.bmp tile_1.bmp
```

Because of them, we can remove them with the file-deduping tool [fdupes](https://code.google.com/p/fdupes/)!

```
fdupes -dN tiles/
```

We just went from 320 to 27! Now to stick them back together into a tileset that I can load into my map editor. I choose 6x5 and use ImageMagick's montage command.

```
montage -border 0 -geometry +0+0 -tile 6x5 tiles/tile* tileset.png
```

The result is a perfect Final Fantasy Adventure tileset.

![tileset](/content/images/2014/Dec/tileset.png)

_Notes:
1. OSX users can install both ImageMagick and fdupes with [homebrew](http://brew.sh/).
2. The multiple water tiles are not a mistake. Look closely, those are distinct tiled._
