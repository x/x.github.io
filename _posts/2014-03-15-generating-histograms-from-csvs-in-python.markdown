---
layout: post
title: Generating Histograms from CSVs in Python
date: 2014-03-15 01:39:36.000000000 -04:00
---
The other day I was looking at a bunch of data I had generated to csv files and I needed a way to programmatically create some histograms, preferably in python. I googled, and surprisingly, nothing came up. So, here's my example code using matplotlib.

<script src="https://gist.github.com/x/9552782.js"></script>

If you're looping through the data and generating a bunch of different histograms (like I was), just be sure the clear the plot with ```plt.clr()```.
