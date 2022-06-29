# Python Jetraw onboarding docs.

This is a proposal to add a better UX for customers when they are getting started with pyJetraw.

---

# Introduction

Welcome to the official documentation of pyJetraw and pyDpcore, the Python wrappers around the official Jetraw and Dotphoton core distributions. The Jetraw algorithm is responsible for compressing images after they were prepared by Dotphoton Core. The following guide should help you get started with it. If you have any feedback, issues or concerns, [feel free to create an Issue here](https://github.com/Jetraw/pyJetraw/issues) and we will do our best to resolve it as fast as we can!

> Note: Throughout this documentation, Dotphoton Core will be referred to as DPCore.
> 

---

# Installation

## Prerequisites

The prerequisites installing pyJetraw are quite simple, all you need is the following:

- Jetraw UI installed with a valid license or a CLI. [Download here!](https://www.jetraw.com/downloads/software)
- Python version 3.7.0 or higher. [Download here!](https://www.python.org/downloads/)

## Step 1: Install the packages

Download both `.whl` files that contain the packages [pyJetraw](https://github.com/Jetraw/pyJetraw/releases/download/21.06.23.2/JetRaw-0.9.1-py3-none-any.whl) and [pyDpcore](https://github.com/Jetraw/pyDpcore/releases/download/21.06.23.1/DPCore-0.9.0-py3-none-any.whl), and put them in a place known to you on your computer. [You can find both of them here!](https://www.jetraw.com/downloads/software)

Then, open a terminal, navigate to where both of these files are, and install them using `pip` :

`pip install DPCore-x.y.z-py3-none-any.whl`****

`pip install JetRaw-x.y.z-py3-none-any.whl`****

pyJetraw also depends on a package called `numpy`, so make sure it is installed on your system:

`pip install numpy`    

A useful tool we will use to load unprepared [TIFF images](https://en.wikipedia.org/wiki/TIFF), is called `tifffile` , so make sure you install it now to follow along:

`pip install tifffile`

> Note: On Linux, if `pip` does not help, try running the above commands using `pip3`
> 

## Step 2: Locate Jetraw & DPCore Libraries.

If you are on Windows, please skip this step.

pyJetraw and pyDPCore need access to the dynamic libraries that come included with Jetraw UI, which are located in your application folder, for example, on MacOS, they are in usually found in the `/Applications/Jetraw\UI.app/Contents/jetraw/lib/` folder, on Linux, this may differ.

So that pyJetraw and pyDPCore know where the C++ libraries are located, you need to explicitly tell them where they are. You do this by setting an environment variable, remember that if you are using an IDE to run python, you will need to launch it from the Terminal. If not the environment will not be correctly configured:

### On Linux

`export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/path/to/jetraw/lib/`

> Note: It is recommended to put this inside of your `.bashrc` so you don't have to set this each time you want to work with pyJetraw and pyDPCore.
> 

### On Mac

`export DYLD_FALLBACK_LIBRARY_PATH=$DYLD_FALLBACK_LIBRARY_PATH:/path/to/jetraw/lib/`

> Note: It is recommended to put this inside of your `.zshrc` so you don't have to set this each time you want to work with pyJetraw and pyDPCore.
> 

This is all you need to get started, now it's time to write some code!

---

# Hello, Jetraw.

A basic workflow with Jetraw and DPCore might look something like this:

1. Load an unprepared, uncompressed [TIFF File.](https://docs.fileformat.com/image/tiff/)
2. Prepare it using `dpcore` .
3. Compress it using `jetraw` .
4. Save the compressed image.

If you do not know what a TIFF file is, [read more about it here!](https://docs.fileformat.com/image/tiff/)

This is the code, do not worry if you do not understand  every single bit of it, we will go through this together:

```python
# simple_workflow.py

import jetraw
import dpcore
import tifffile
import numpy as np
import locale

# Set locale to prevent encoding issues.
locale.setlocale(locale.LC_ALL, locale.getlocale())

# Load an unprepared image.
image = tifffile.imread("my_unprepared.tif").astype(np.uint16)

# Load calibration file.
dpcore.load_parameters("path_to_calibration_file/calibration.dat")

# Prepare loaded image.
dpcore.prepare_image(image, "calibration_identifier")

# (optional) Save the prepared file to compress it later.
# tifffile.imsave("my_prepared.tiff", data=image)

# Compress and save file.
jetraw.imwrite('my_compressed.tif', image, description="Python Jetraw Tests")
```

In the first and second line, we are importing `jetraw` and `dpcore` to gain access to the functions that prepare and compress your image. To read in unprepared .tiff or .tif files, we import the `tifffile` module we installed earlier. To convert the loaded image to an appropriate format, we import `numpy as np` into our project. Finally we import `locale` which helps us prevent encoding issues the C++ libraries might have. We do that like so:

```python
locale.setlocale(locale.LC_ALL, locale.getlocale())
```

Then, it's time to finally load an image. Loading an image is quite simple, all you have to to is call the `tifffile.imread()` function, supply it with a path to an unprepared TIFF file and it will give you back a `numpy.ndarray`, [which is a multidimensional container structure](https://numpy.org/doc/stable/reference/arrays.ndarray.html#:~:text=An%20ndarray%20is%20a%20(usually,the%20sizes%20of%20each%20dimension.) found in the `numpy` module we imported earlier. To enforce a data format that `dpcore` knows how to work with, we cast that array to the type `np.uint16` , in the end it all looks like that:

```python
image = tifffile.imread("path/to/tiffile.tiff").astype(np.uint16)
```

Next, we have to load the calibration file. Since all cameras are different in some aspect or the other, they all have their quirks and things you need to consider when you handle images taken by that camera, therefore, in order for `dpcore` to know for sure how to handle your .tiff files, you need to obtain a calibration file of the camera that took the images you want to process. If you did not receive one in your testing kit, [you can find one here.](https://github.com/Jetraw/pyDpcore/blob/master/pco_3a2dd3a.dat) Moving on, to load that calibration file you just call the `.load_parameters()` function and pass in a path to your calibration file:

```python
dpcore.load_parameters("path/to/calibration.dat")
```

We're almost there! Now it's time to prepare the TIFF files so `jetraw` knows how to compress them. This is a super trivial task as the only thing you need to do is pass in the image you loaded and a so-called calibration identifier to a function called `prepare_image`, where a calibration identifier is what tells `dpcore` what settings were used when taking the pictures. 

## How to find Calibration Identifiers?

### Option 1: Using the Jetraw U

The easiest approach is opening Jetraw UI, loading in the calibration file and choosing the different calibration identifiers that exist.

First, open Jetraw UI and click on 'Devices':

![Screenshot 2022-04-11 at 09.55.02.png](https://github.com/Jetraw/Jetraw/blob/doc/doc/figures/Screenshot_2022-04-11_at_09.55.02.png)

Then, load your calibration file by clicking 'INSTALL DAT FILE':

![Screenshot 2022-04-11 at 09.55.15.png](https://github.com/Jetraw/Jetraw/blob/doc/doc/figures/Screenshot_2022-04-11_at_09.55.15.png)

Finally, if all goes well, at the top, a dropdown will appear, listing all the calibration identifiers (left column) and their respective settings:

![Screenshot 2022-04-11 at 09.56.01.png](https://github.com/Jetraw/Jetraw/blob/doc/doc/figures/Screenshot_2022-04-11_at_09.56.01.png)

### Option 2: Using the CLI

If you have access to a command line application instead of a GUI, another approach you could take is listing them using the `dpcore` cli. On Linux, this assumes you have your `calibration.dat` in the `.config/dpcore` folder. Just type the following into your terminal:

```bash
$ dpcore --list-ids

# Output should look something like this
61005114_1_86000000_0
  pco.edge 5.5m USB rolling shutter
  Rolling shutter, Pixel rate 86.0 MHz
61005114_1_86000000_1
  pco.edge 5.5m USB rolling shutter
  Rolling shutter, Pixel rate 86.0 MHz, Noise filter on
61005114_2_160000000_0
  pco.edge 5.5m USB rolling shutter
  Global shutter, Pixel rate 160.0 MHz
61005114_2_160000000_1
  pco.edge 5.5m USB rolling shutter
  Global shutter, Pixel rate 160.0 MHz, Noise filter on
61005114_4_86000000_0
  pco.edge 5.5m USB rolling shutter
  Global reset, Pixel rate 86.0 MHz
61005114_4_86000000_1
  pco.edge 5.5m USB rolling shutter
  Global reset, Pixel rate 86.0 MHz, Noise filter on
```

Please copy the one you find to be appropriate for your images and go back to your code. Now you have all you need to prepare an image:

```python
# Replace second parameter with any of your choice.
dpcore.prepare_image(image, "61005114_1_86000000_0")
```

Optionally, if you just want to prepare the image, you would be done, then you could just save it like this:

```python
# (optional)
tifffile.imsave("my_prepared_image.tiff", data=image)
```

To compress your image and save it as a file, you can use the very convenient `jetraw.imwrite()` function:

```python
jetraw.imwrite("my_compressed_image.tiff", image, description="Python Jetraw Test")
```

If you only want to compress it and NOT save it to a file, you do that by calling `jetraw.encode(image)` instead of `.imwrite()` like so:

```python
# (optional) 
encoded_image = jetraw.encode(image)
```

This will return a 1D buffer with the width and height of the image taking up the first eight bytes of the buffer. (4 bytes - width, 4 bytes - height)

That's about it, fairly easy! Now you know the basics of how to prepare and compress using Jetraw and DPCore!

---

# Decompression

Compressing the images is just for storage purposes, you can not do much with them in their compressed state, therefore, to use your images, you have to decompress your TIFF files again.

To do that you just need to have access to the buffer of your compressed TIFF File and call the `jetraw.decode()` function on it. To load the file, we can use `jetraw.imread()` to read a compressed file and decode it immediately or decompress an image buffer using `jetraw.decode()` . The latter returns a decoded image buffer will look identical to the original.   A program the decompresses an image might look like this:

```python
import jetraw
import numpy as np
import tifffile # We still need this for saving the file.

# Read image and compress it immediately.
image = jetraw.imread("my_compressed.tif")

# Decompress the image having only an image buffer.
image = ... # Those are the bytes of the encoded image
decoded = jetraw.decode(image)

# Save it!
tifffile.imsave("my_decompressed_image.tiff", image)
tifffile.imsave("my_decompressed_image_buffer.tiff", decoded)
```

That's about it! 

---

# Interacting with compressed files.

Although above was mentioned that you can not do much with compressed files, you still can access them and get information about them using the `jetraw.TiffReader` class:

```python
tiff = jetraw.TiffReader("some_file.tiff")

print(tiff.pages)  # Amount of pages in TIFF file.
print(tiff.width)  # Width of a page.
print(tiff.height) # Height of a page.

tiff.close()

# Optional 'with' Syntax (recommended)
with jetraw.TiffReader("some_file.tiff") as tiff:
    print(tiff.width)
		# ...
```

> Note: Quite conveniently, this also works with files that are not compressed by Jetraw.
> 

---

# Working with multi-page TIFF files.

The structure of a TIFF file allows it to contain more than one image, generally referred to as “pages”,  in order to compress, prepare and interact with multiple-page TIFF files, some things need to be done a little differently.

## Getting the amount of pages in a multi-page TIFF file.

There are multiple ways to determine the amount of pages found in a TIFF file. If you load an image using `tifffile.imread()` you can get the amount of images by accessing the first element of the shape of the `ndarray` that gets returned:

```python
>>> import tifffile
>>> i = tifffile.imread("test.tiff")
>>> i.shape[0]
(10, 2560, 2160)
```

 

> Note: This will only work if your TIFF file *has* multiple pages, otherwise the `i.shape[0]` will evaluate the width of the image.
> 

Alternatively, as seen above, you can just use the `jetraw.TiffReader` .

## Preparation and Compression.

To prepare and compress multi-page TIFF file you need to loop through the pages and process them individually:

```python
import jetraw, dpcore, tifffile

image = tifffile.imread("many_pages.tiff")

for page in range(image.shape[0]):
    dpcore.prepare_image(image[page][:][:])

# If you are working with Buffers and NOT saving to a file.
# Replace .encode() with .decode() and you would be decompressing files.
encoded_pages = []
for page in range(image.shape[0]):
    encoded_pages.append(jetraw.encode[page][:][:])

# (option 1) Saving to a file still works as is!
jetraw.imwrite("many_pages_compressed.tiff", image, description="Hi")

# (option 2) Use a Writer Object.
with jetraw.TiffWriter("many_pages_compressed.tiff", "Hi") as jw:
    for page in range(image.shape[0]):
        jw.write(image[page, :, :])
```

> Note on Writer and Reader Objects: They exist for the sake of convenience and better control over individual pages.
> 

# You made it!

As you can see, using Jetraw and DPCore through Python is a breeze, it's super easy, convenient and fast (because the heavy lifting is done by the high-performant C++ libraries). If you have any questions or did not understand something correctly, feel free to post an issue, we will do our best to get back to you!
