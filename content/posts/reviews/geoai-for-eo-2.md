---
title: Ch2 - GeoAI for EO Imagery
date: 2026-06-25 08:00:00 -0500
font_mode: serif
categories:
  - reviews
tags:
  - books
  - testbooks

summary:
---

This chapter is about image pre-processing: going from the raw signal on the ground to the actual data contained in the satellite image. There was a ton in here that was deeply non-trivial and unfamiliar to me... it was a challenging start to the book.

## Definitions

- **Radiance** the amount of radiation given off by a surface. Units are usually in $W sr^{-1} $
- **NIR** Near Infrared. 750 nm to 1400 nm.
- **SWIR** Short wave infrared. 1400 nm to 3000 nm.

## What a remote sensing satellite does

> I've had to back up pretty far, to make sense of this chapter. A lot of this information comes from [here](https://sentiwiki.copernicus.eu/web/s2-mission)

The primary function of a (remote sensing) satellite is to measure _radiance_: the amount of radiation coming from an area.[^1] They do this with a _photodetector_, which, in my understanding, is composed of:

1. A light-sensistive photodiode, which recieves radiation and releases electrons via the [photoelectric effect](https://en.wikipedia.org/wiki/Photoelectric_effect).
2. A series of transistors, responsible for sucking up and amplifying those freed electrons (as well as resetting the photodiode and other, more complicated stuff). The output of this portion is a voltage that varies with the number of electrons released by the photodiode.
3. An analog to digital converter, which converts the voltage into a binary number (Called a Digital Number, DN). The length of this binary number (the _bit depth_) is determined by the sensor. It seems like most common sensors use 12 bits.

What you do with this number, and how it compares to the actual physical quantity you measure, is an anlysis problem for later.

## Major differences between camera types

While the cameras on satellites and the cameras tourists hold both operate _generally_ according to the same general model above, there are some differences:

### Sensor array configuration
The cameras on your phone use a 2D grid of sensors (a _staring_ array). They are (barring your own self control) stable when you take the picture, and capture the whole 2D image at once. Some satellites operate like this, but most use either _pushbroom_ or _whiskbroom_ sensor configurations. The wikipedia articles for these sensors are short, and show some really helpful animations ([pushbroom](https://en.wikipedia.org/whttps://en.wikipedia.org/wiki/Whisk_broom_scanneriki/Push_broom_scanner), [whiskbroom](https://en.wikipedia.org/wiki/Whisk_broom_scanner)).

Note that with pushbroom and whiskbroom, we don't get discrete images so much as a single long, skinny image, representing a strip of the surface of the earth. It's kinda like taking a pano on your phone that never ends. These long strips are often chopped up into single images for downstream processing. For example, landsat images a continuous swath of the earth around 180km in width, but for downstream processing chops it up into 185km x 180km (or something along those lines) individual images.

### Satelittes often capture more than just RGB
A traditional digital camera captures RGB by using a [Bayer filter](https://en.wikipedia.org/wiki/Bayer_filter). Satellites capture a whole range of bands. For instance, the Sentinel-2 program has 13 different bands, 3 of which are RGB. Common bands include:

- Ultraviolet/blue. Commonly used for aerosol
- RGB
- Near infrared (NIR). Often used for vegetation health.
- Short wave infrared (SWIR). Often used for water/moisture content.
- Thermal infrared.
- Pancromatic. Captures a wide range of wavelengths at higher resolution.

For remote sensing tasks it is common to use combinations of these bands. I've found [this site](https://gisgeography.com/spectral-signature/) has a good overview of common wavelengths and their different use cases, and they also have an article on the use cases of [different band combinations](https://gisgeography.com/sentinel-2-bands-combinations/). There is an interesting history of the different LANDSAT bands [here](https://science.nasa.gov/mission/landsat/spectral-bands-and-applications/)

Often, data acquisition for the different bands involves shuffling around the incoming light to different sensors using series of mirrors, and it is frequently the case that different bands are captured at different resolution.

Some satellites even have cameras that can capture *hundreds* of different bands. This is called *hyperspectral imaging*.

## 17 minutes and the maximum period of observation is 32 minutes.

Light reflected up to the MSI instrument from the Earth and its atmosphere is collected by a three-mirror (M1, M2 and M3) telescope and focused, via a beam-splitter, onto two Focal Plane Assemblies (FPAs): one for the ten VNIR wavelengths and one for the three SWIR wavelengths.

The output of this array of

## What the sensor actually picks up

[^1] I suppose this is the purpose of a camera in general haha.
