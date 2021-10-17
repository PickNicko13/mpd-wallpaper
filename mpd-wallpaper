#!/bin/bash

# disable output if glob not matched
shopt -s nullglob

# init arrays
extensions=( .png .jpg .jpeg .webp .PNG .JPG .JPEG .WEBP )
covernames=({Cover,cover,Cover1,cover1,Front,front}.{png,jpg,jpeg,webp,PNG,JPG,JPEG,WEBP})
previous_cover_file=""

set_cover(){
	cover_file=""
	# search for well-named covers
	for covername in ${covernames[*]}; do
		if [ -f "$path$covername" ]; then
			cover_file="$path$covername"
			echo $(date '+%H:%M:%S') found artwork in: "$cover_file"
			break
		fi
	done
	# if not found, search for any covers as files
	if [ -z "$cover_file"  ]; then
		# this part is retarded
		potential_covers=""
		for ext in ${extensions[*]}; do
			potential_covers=( "$path"*"$ext" )
			if (( ${#potential_covers[@]} )); then
				cover_file=${potential_covers[0]}
				echo $(date '+%H:%M:%S') found artwork in: "$cover_file"
				break
			fi
		done
	fi
	# if not found, try to extract an embedded one
	if [ -z "$cover_file"  ]; then
		# if no cover art found even here
		if ! ( kid3-cli -c get "$track_file" | grep -q Picture ); then
			echo $(date '+%H:%M:%S') "fix needed: \"$track_file\""
			echo $(date '+%H:%M:%S') "fix needed: \"$track_file\"" >> ~/broken.txt
			feh --no-fehbg --bg-center "$1"
			return 1
		fi
		previous_cover_file=""
		echo $(date '+%H:%M:%S') found embedded artwork in: "$track_file"
		kid3-cli -c 'get picture:/tmp/new_embedded_cover.jpg' "$track_file"
		if [ -f /tmp/embedded_cover.jpg ]; then
			if [ $(du -b -s /tmp/embedded_cover.jpg | cut -f1) \
				== $(du -b -s /tmp/new_embedded_cover.jpg | cut -f1) ] ;then
				echo $(date '+%H:%M:%S') same embedded cover
				rm /tmp/new_embedded_cover.jpg
				return 0
			fi
		fi
		mv /tmp/new_embedded_cover.jpg /tmp/embedded_cover.jpg
		cover_file="/tmp/embedded_cover.jpg"
	fi

	# by this moment cover_file must not be empty
	if [ -z "$cover_file"  ]; then
		echo $(date '+%H:%M:%S') "WTF: \"$track_file\""
		echo $(date '+%H:%M:%S') "WTF: \"$track_file\"" >> ~/broken.txt
		# feh --no-fehbg --bg-center "$1"
		# uncomment the feh line and comment the xwallpaper one
		# if you want to use a fallback image that's not in png or jpg format
		xwallpaper --center "$1"
		return 2
	fi

	# avoid redoing what's already done
	if [ "$previous_cover_file" == "$cover_file" ]; then
		echo $(date '+%H:%M:%S') "same cover art"
		return 0
	fi
	previous_cover_file="$cover_file"

	# scale and convert to bmp
	# change the values to the appropriate ones

	magick "$cover_file" -filter sinc -resize $resolution^ \
	   -gravity Center -crop $resolution+0+0 -blur 0x16 -level 0%,200% /tmp/low_layer.bmp
	magick "$cover_file" -filter sinc -resize $subresolution \
		-background none -compose Copy -gravity south -extent $resolution /tmp/high_layer.bmp
	magick /tmp/{low,high}_layer.bmp -background None -layers Flatten \
		-define png:compression-level=0 /tmp/background.png
	rm /tmp/{low,high}_layer.bmp

	# finally, set the background
	xwallpaper --center "/tmp/background.png"
}

# not enough arguments given
if [ -z "$2" ]; then
	echo "	Usage:
	$0 fallback/image bar_height
	e.g.: $0 \$HOME/Pictures/Wallpapers/GiTS/1.png 24"
	exit 1
fi
# file doesn't exist
if ! [ -f "$1" ]; then
	echo couldn\'t open \""$1"\"
	exit 1
fi

resolution=$( xdpyinfo | grep -P -o '[0-9]*x[0-9]* pixels' | cut -f1 -d' ' )
subresolution=${resolution%x*}x$(( ${resolution##*x} - $2 ))
musicpath=$( grep 'music_directory' $HOME/.mpd/mpd.conf | cut -f2 -d' ' | sed "s+\"++g; s+~+$HOME+" )
echo -e "resolution: $resolution\nsubresolution: $subresolution"

while true; do
	newtrack=$(mpc -f '%file%' current)
	if ! [ "$newtrack" == "$track" ]; then
		echo $(date '+%H:%M:%S') track changed to $newtrack
		# comment the line below if you don't want a notification on song change
		kunst --size 60x60 --silent
		track="$newtrack"
		track_file="$musicpath/$track"
		path="${track_file%/*}/"

		starttime=$(date +'%s.%N')
		set_cover
		echo took $(echo "scale=9;$(date +'%s.%N')-$starttime" | bc)s
	# else
	#	echo $(date '+%H:%M:%S') track not changed
	fi
	mpc idle > /dev/null
done
