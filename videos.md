# Video Manipulation

I don't tend to do much video manipulation and conversions, as I mainly deal with photography. However,
I do some basic editing of videos for my drone.

If your needs are basic you may find some of the following useful.

## View information about video files

It is often useful to see the codec, frame rate, bitrate etc. of a video file. You can get some information by right
clicking and looking at the properties tab. However, if you need the full set of info then it is better to use ffprobe.

ffprobe is included with ffmpeg:

	sudo dnf install ffmpeg

Then you can use the following to view file information:

	ffprobe -v quiet -print_format json -show_format -show_streams -print_format json "~/Videos/yourfile.mp4"

## OpenShot Profiles

There are various excellent free and opensource video editors out there (Kdenlive, Shotcut, Flowblade etc.), but for the moment
I have settled on OpenShot as it is (to me at least) the most intuative.

	sudo dnf install openshot

When coming to output your video, this is controlled by profiles, which are defined in the following location:

	/usr/lib/python3.9/site-packages/openshot_qt/presets/

I have found that for some reason it is not possible to change the bitrate beyond low, med or high which is hardcoded
in these profile files. You can of course create your own profile to get around this problem. Just make a new file. 

## HEVC(H.265) vs H.264

If you need to output your project there are quite a few formats available, and it can be quite confusing.

The most commonly used format is H.264 (.mp4). This is a great format to use and generally guarentees compatibility. 

HEVC is a more modern format still output as ".mp4". It uses an improved algorithm to allow equivalent quality at a
lower bitrate. What this means is you can get smaller file sizes without loss of quality, when compared to H.264.

As a general rules you can use 50-75% of the bitrate to produce an equivalent quality HEVC file.

HEVC also has the advantage of 10-bit rather than 8-bit in H.264 (although H.264 can use 10-bit it is not widely supported).

The long and short is:

	- use HEVC when you can, as at the same bitrate it is superior to H.264
	- if you need wide compatibility use H.264  

One downside of HEVC is that due to the improved algorithm it takes longer to encode the file.

## Using your graphics card

When encoding video it is always better to utilise your graphics card if you can. It is just quicker.

Another nice feature of OpenShot is that the profile you pick to encode the files will specifically state whether it
uses the CPU or a GPU, making it easier to ensure you pick the correct one.

Graphics card profiles are labelled bright green to make it more obvious:

	- VA-API - this stands for Video Acceleration API, this is what I use with my AMD GPU
	- NCENC - Nvidia video encoding
	- QSV - Intel Quick Video Sync

If you want to double check that your AMD GPU is being used, then you can display the graphics card usage using radeontop:

	sudo dnf install radeontop

