---
title: "Working with Pathology"
teaching: 60
exercises: 30
---

:::::::::::::::::::::::::::::::::::::: questions 

- What are some open libraries can be used with pathology?
- How are pathology slides different from a photo?
- What is the structure of a pathology slide?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Describe some open source tools and libraries for pathology images
- Discuss  common file formats for pathology slides
- Explain tiling and pyramidal file formats
- Show basic code in openslide

::::::::::::::::::::::::::::::::::::::::::::::::

## Introduction

Pathologists deal with multiple types of images. Some pathologists may use photography when recording gross pathology data. The processing of digital photography is a straightforward application of general image processing. However the workhorse type image of pathology is the histopathological image, and this is very different from a photograph. In this lesson we will cover how such an image is set up and differs from a simple multi-channel color image like a modern digital camera often produces. 

## File formats

In terms of histopathological images, you may find them in several file formats. Not every library or tool for such imaging works with every format. Below is a table with several file formats.


| Format Name |Extension|Notes|  More info|
| ----------- |---------|-----|-----------|
| TIFF     |.tiff|tag image file format| can be tiled |https://www.loc.gov/preservation/digital/formats/fdd/fdd000022.shtml|
| OME-TIFF       |.ome.tiff, ome.tif, .ome.tf2, .ome.tf8 or .ome.btf | includes XML and may be multiple TIFFs|https://brainder.org/2012/09/23/the-nifti-file-format/|
| SVS         | .svs    | Scanned Virtual Slide  |https://www.dicomstandard.org/  |
| DICOM       |.dcm or none|supports an included WSI file|https://dicom.nema.org/dicom/dicomwsi/|



 The table above is far from complete because any vendors have thier own file format, however the community is consoludating towards shared file formats that are not vendor specific. The table is organized in a manner of ascending complexity. A simple general TIFFfile is very close to a simple raster image. Such a file for a high resolution image would be quite large and potentially hard to manage based on size alone.Tiling in a way to more efficiently manage huge images. 
 Tiling  approaches will be very familiar to anyone who comes from the world of GIS, mapping or digital geography. Maps provide a really obvious way to think about the advantages of pyramidal tiling. Imagine if every time you wanted to get a map with directions, you needed to load a detailed map of the entire world, literally. It would be incredibly ineffiient and taxing on your computer or cellphone. It's much more efficient to just load what you need. There are many ways this can be done but practically some variation of pyramidal tiling is usually used. The same is true with tiled pathology images.
 
 
 Tiled images are made in a pyramidal structure, and at each level of the pyramid after zero there is a lower resolution downscaled version of the image. Each level of the pyramid has tiles, smaller parts ofthe image that are quilted into the whole image. Creating images in this way allows much more efficient access and management of images because you can load and display images at only one level. 
 An OME-TIFF file can be one or more tiled TIFFs and some XML in the header. An SVS file is esssentially a tiled TIFF image that has a few additional things like overview image. Various types of pathology files can be somehow packed into a DICOM which has it's own file specification. 

## Libraries

There is actually very little in the way of open pure python libraries that deal with pathology images. There are however popular libraries that have binders or other tricks that allow you to process pathology images in Python. Probably the most popular is openslide.



:::::::::::::::: callout

### Popular open libraries for pathology:

- QuPath
- Openslide

::::::::::::::






#### Reading TIFF histopathology Images

[Openslide](https://openslide.org/) is actually written in C, but you can also use Java or Python to work with is due to bindings.To run the code you will need to work in the `pathy` environment created with the environment_pathology.yml file.

```python
import openslide
from openslide import open_slide
import numpy as np
import matplotlib.pyplot as plt
```

After importing the needed libraries we can now load a slide, and print out it's properties.

```python
file_path = 'C:/Projects/medical-image-processing-materials/tubhiswt-3D/CMU-1.tiff'

slide_in = open_slide(file_path)

slide_in_props = slide_in.properties
print(slide_in_props)
```

When we printed the properties we can see we are dealing with a tiled slide because we have properties like levels and tile height and width. These may not be the only properties we care about. We most likely may care about mapping this image back to real world sizes. We have a property that tells us the microns per pixel. Let's look:

```python
print("Pixel size of X in um is:", slide_in_props['openslide.mpp-x'])
print("Pixel size of Y in um is:", slide_in_props['openslide.mpp-y'])
```

```output
Pixel size of X in um is: 1000
Pixel size of Y in um is: 1000
```

:::::::::::::::::::::::::::::::::::::::  challenge

## Challenge: Find the real world size of an image

There is a property of the loaded image which is (width, height) tuple for level 0 of the image. The property is called `dimensions`. Calculate the real world dimensions of our image in microns. 

:::::::::::::::  solution

## Solution

```python
# Retrieve the dimensions of the image at the highest resolution (level 0)
width, height = slide.dimensions

# get the spatial resolution of the slide (in microns per pixel)
# get the spatial resolution in microns per pixel for the X and Y directions
microns_per_pixel_x = slide.properties.get(openslide.PROPERTY_NAME_MPP_X)
microns_per_pixel_y = slide.properties.get(openslide.PROPERTY_NAME_MPP_Y)

# convert to float in case they are strings
microns_per_pixel_x = float(microns_per_pixel_x)
microns_per_pixel_y = float(microns_per_pixel_y)

# calculate real-world dimensions of the slide in microns
real_world_width = width * microns_per_pixel_x
real_world_height = height * microns_per_pixel_y

# print the results

print(f"Real-world dimensions in microns: {real_world_width} microns (width), {real_world_height} microns (height)")
```

```output
Here the output
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::


As you might guess loading this huge image is pointless if we just want a small image that gives us a general idea. We can use openslide to make a thumbnail.


```python
# get a thumbnail of the image and pop it out on your computer
slide_in_thumb_600 = slide_in.get_thumbnail(size=(600, 600))
slide_in_thumb_600.show() # 
```

```output
Add image here?
```

We could also store off our thumbnail ( or any part of the image for that matter )as a numpy array.

```python
# convert thumbnail to numpy array, and plot it
slide_in_thumb_600_np = np.array(slide_in_thumb_600)
plt.figure(figsize=(8,8))
plt.imshow(slide_in_thumb_600_np)  
```

```output
```

We can also explore a bit about how the image is tiled.

```python

# get slide_in dims at each level of the pyramid at various levels
dims = slide_in.level_dimensions

num_levels = len(dims)
print("Number of levels in this image are:", num_levels)

print("Dimensions of various levels in this image are:", dims)

# understand by how much are levels downsampled from the original image
factors = slide_in.level_downsamples
print("Each level is downsampled by an amount of: ", factors)
```



