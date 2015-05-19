# LimeRPi2-kodi
Limelight on OpenELEC. (Kodi)

Uses [the limelight project](https://github.com/irtimmer/limelight-embedded).
Also posted on [the OpenELEC forums.](http://openelec.tv/forum/12-guides-tips-and-tricks/76298-how-to-setup-limelight-on-the-raspberry-pi#137002)

Installation:
--------------
1. Put [Java for ARM](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-arm-downloads-2187472.html) on the pi in /storage/java and test it.
```
mkdir -p /storage/java
...extract the java archive and put the contents of the java_1. folder in the /storage/java folder...
/storage/java/bin/java -version
```

2. Create the limelight folder.
```
mkdir -p /storage/limelight
cd /storage/limelight
```

3. Pair the pi with the computer. (substitute 192.168.0.150 with the IP of your desktop)
```
/storage/java/bin/java -jar /storage/limelight/limelight.jar pair 192.168.0.150
```

4. Run the following command to create the script to run limelight in /storage/limelight/run.sh
Again, substitute 192.168.0.150 with the IP of your desktop.
```
cat >/storage/limelight/run.sh <<EOL
#!/bin/sh

if [[ ! $LD_LIBRARY_PATH == *"/storage/limeligt"* ]]; then
  echo 'Changed library path'
  export LD_LIBRARY_PATH=/storage/limelight:$LD_LIBRARY_PATH
fi

if ! lsmod | grep "snd_bcm2835" &> /dev/null ; then
  echo 'Loaded sound module'
  modprobe snd_bcm2835
fi

echo 'Exiting Kodi..'
systemctl stop kodi

echo 'Starting limelight..'
/storage/java/bin/java -jar /storage/limelight/limelight.jar stream 192.168.0.150

echo 'Finished. Firing Kodi back up..'
systemctl start kodi
EOL
```

5. Run the following command to create the update script for you.
```
cat >/storage/limelight/update.sh <<EOL
function version { echo "$@" | gawk -F. '{ printf("%03d%03d%03d\n", $1,$2,$3); }'; }
 
function downloadFile {
	echo "$1" | egrep -o "/irtimmer/moonlight-embedded/releases/download/v([0-9]\.*)+/$2" | wget -q --base=http://github.com/ -i - -O "$2"
}
 
function updateMoonlight {
	FILE=version
	releases=`curl -s -L https://github.com/irtimmer/moonlight-embedded/releases/latest`
	current_version=`cat $FILE`
	latest_version=`echo "$releases" | egrep -o "/releases/download/v([0-9]\.*)+/" | egrep -o "v([0-9]\.*)+" | cut -c 2- | head -n 1`
 
	if [ ! -f "$FILE" ] || [ "$(version "$latest_version")" -gt "$(version "$current_version")" ]; then
		echo "Updating moonlight to $latest_version"
		downloadFile "$releases" libopus.so
		downloadFile "$releases" limelight.jar
		echo "$latest_version" > "$FILE"
		return 0
	else
		echo "No update necessary, at latest version. ($latest_version)"
		return 1
	fi
}
 
updateMoonlight
EOL
```

6. And finally, make the scripts we just created executable.
```
chmod +x /storage/limelight/run.sh
chmod +x /storage/limelight/update.sh
```
7. Now run `/storage/limelight/update.sh` to download the latest files.

Finished! Whenever you want to play games using limelight, just run /storage/limelight/run.sh.
You can even create a link in Kodi to make it as seamless as possible.
To update limelight, just run /storage/limelight/update.sh

