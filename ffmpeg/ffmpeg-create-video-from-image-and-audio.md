# ffmpeg create video form audio and image

Combine image and audio file into a video

```
ffmpeg \
-hide_banner \
-stats -v panic \
-loop 1 \
-r 1 \
-i img.jpg \
-i audio.m4a \
-c:v libx264 \
-preset veryfast \
-tune stillimage \
-crf 17 \
-c:a copy \
-vf "yadif=0:-1:0, scale=720:trunc(ow/a/2)*2" \
-pix_fmt yuv420p \
-movflags +faststart -f mp4 \
-shortest \
video-$(date +"%Y-%m-%d-%H-%M-%S").mp4
```

## still-video script

Script to combine image and audio into a video file

bash shebang for unix

```
#!/usr/local/bin/bash
```

if you are using linux or mac osx, change the shebang to:

```
#!/bin/bash
```

still-video script

```
#!/usr/local/bin/bash

# create a video from an jpg image and m4a audio file

# script checks
script_usage='Too many arguments passed to script'
script_purpose='Create a video from a jpg image and m4a audio file'
image_type='Invalid input, please select a jpg image file'
audio_type='Invalid input, please select a m4a audio file'
videofile="$HOME/Desktop/video-$(date +"%Y-%m-%d-%H-%M-%S").mp4" # default recording directory

# check number of args passed to script
[[ $# -eq 0 ]] || { printf "%s\n" "$script_usage"; exit; }

# display script purpose when the script is run 
printf "%s\n" "$script_purpose"

# prompt user to select a image file
read -erp "enter the path to a jpg image file: " image
[[ "$image" =~ \.jpg?$ ]] || { printf "%s\n" "$image_type"; exit; }

# prompt user to select a m4a audio file
read -erp "enter the path to a m4a audio file: " audio
[[ "$audio" =~ \.m4a?$ ]] || { printf "%s\n" "$audio_type"; exit; }

# record ffmpeg function
record () {
ffmpeg \
-hide_banner \
-stats -v panic \
-loop 1 \
-r 1 \
-i $image \
-i $audio \
-c:v libx264 \
-preset veryfast \
-tune stillimage \
-crf 17 \
-c:a copy \
-vf "yadif=0:-1:0, scale=720:trunc(ow/a/2)*2" \
-pix_fmt yuv420p \
-movflags +faststart -f mp4 \
-shortest \
"$videofile"
}

# run record function to combine image and audio into a video
printf "%s\n" "Combining $image and $audio into video file"
record && printf "%s\n" "Created $videofile"
```
