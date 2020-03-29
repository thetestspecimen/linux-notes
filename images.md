# Image Manipulation

It is often required to compress and resize images.

With the use of ImageMagick this can be done via the commandline in batches.

ImageMagick: performs all image manipulation

parallel: allows faster batch processing by running jobs in parallel

## Install

	sudo dnf install ImageMagick parallel

## WARNING

**The mogrify command is destructive.** For example if you run the following command inside a folder full of JPG images it will overwrite the originals!

	find . -type f | egrep "*.jpg" | parallel mogrify -quality 100 -filter Lanczos -resize 640x640^ -format jpg {/}

You would lose the original full size images forever as they are overwritten with 640x640 smaller images!

**BE VERY CARFEFUL, YOU HAVE BEEN WARNED**

However, this is no problem:

	find . -type f | egrep "*.png" | parallel mogrify -quality 100 -filter Lanczos -resize 640x640^ -format jpg {/}

This is because it converts from png to jpg (i.e. the file endings are different), so the originals will not get touched.

## Conversion examples

All the conversion examples should be run from the commandline in the folder where the image files are located.
Outputs will be generated in the same folder

### Lossless conversion from TIF to WebP Lossless

	find . -type f | egrep "*.tif" | parallel mogrify -quality 100 -define webp:lossless=true -define webp:image-hint=photo -define webp:method=6 -format webp {/}

### Conversion from TIF to JPG with resizing

	find . -type f | egrep "*.tif" | parallel mogrify -quality 100 -filter Lanczos -resize 720x720^ -format jpg {/}
	find . -type f | egrep "*.tif" | parallel mogrify -quality 100 -filter Lanczos -resize 480x480^ -format jpg {/}
	find . -type f | egrep "*.tif" | parallel mogrify -quality 100 -filter Lanczos -resize 960x960^ -format jpg {/}
	find . -type f | egrep "*.tif" | parallel mogrify -quality 100 -filter Lanczos -resize 640x640^ -format jpg {/}


### Conversion from TIF to WebP (lossy) with resizing

	find . -type f | egrep "*.tif" | parallel mogrify -quality 100 -filter Lanczos -resize 720x720^ -define webp:lossless=false -define webp:image-hint=photo -define webp:method=6 -format webp {/}
	find . -type f | egrep "*.tif" | parallel mogrify -quality 100 -filter Lanczos -resize 480x480^ -define webp:lossless=false -define webp:image-hint=photo -define webp:method=6 -format webp {/}
	find . -type f | egrep "*.tif" | parallel mogrify -quality 100 -filter Lanczos -resize 960x960^ -define webp:lossless=false -define webp:image-hint=photo -define webp:method=6 -format webp {/}
	find . -type f | egrep "*.tif" | parallel mogrify -quality 100 -filter Lanczos -resize 640x640^ -define webp:lossless=false -define webp:image-hint=photo -define webp:method=6 -format webp {/}

### Notes on the commands

The first part:

	find . -type f | egrep "*.tif" | parallel mogrify

Specifies to find, within the current folder, all files with extension "tif". It also specifies the use of the methods "parallel" and "mogrify", which will carry out the conversion.

- Quality: specifies the compression aggressiveness from 0 to 100
- Filter: when resizing a picture an algorithm needs to be used to reduce the amount of pixels. There are various methods available, but I have chosen the Lanczos method.
- Resize: this tells Imagemagick what size the final image should be. Specifically "720x720^" means the smallest side of the image will be 720 pixels long
- Format: This specifies the output format and can be whatever is necessary

The remaining options that you see for WebP are specific to that particular file format. Depending on the file format output there may be other options available.
Please check the documentation on the ImageMagick website for further details.

## ImageMagick Source

ImageMagick is a hugely powerful tool, which I have barely touched upon in this guide.

I highly recommend visiting the website to see if there are any further features that can be of use:

https://imagemagick.org/index.php