# LimeRPi2-kodi
moonlight on OpenELEC. (Kodi)

Uses [the moonlight project](https://github.com/irtimmer/moonlight-embedded).
Also posted on [the OpenELEC forums.](http://openelec.tv/forum/12-guides-tips-and-tricks/76298-how-to-setup-moonlight-on-the-raspberry-pi#137002)

Installation:
--------------
1. Connect to your Pi over SSH.

2. This version of limelight still needs Java, so let's put it on the pi in /storage/java and test it. (Future versions will not be java anymore, so no worries.)
```
mkdir -p /storage/java
curl -L -O -s https://github.com/HazCod/LimeRPi2-kodi/blob/master/jdk-8u33-linux-arm-vfp-hflt.tar.gz?raw=true
tar xvf jdk-8u33-linux-arm-vfp-hflt.tar.gz
rm tar xvf jdk-8u33-linux-arm-vfp-hflt.tar.gz
mv jdk1.8.0_33/* /storage/java
/storage/java/bin/java -version
```

3. Create the moonlight folder.
```
mkdir -p /storage/moonlight
cd /storage/moonlight
```

4. Pair the pi with the computer. (substitute 192.168.0.150 with the IP of your desktop)
```
/storage/java/bin/java -jar /storage/moonlight/moonlight.jar pair 192.168.0.150
```

5. Run the following command to create the script to run moonlight in /storage/moonlight/run.sh
Again, substitute 192.168.0.150 with the IP of your desktop.
```
cat >/storage/moonlight/run.sh <<EOL
#!/bin/sh

if [[ ! $LD_LIBRARY_PATH == *"/storage/limelight"* ]]; then
  echo 'Changed library path'
  export LD_LIBRARY_PATH=/storage/moonlight:$LD_LIBRARY_PATH
fi

if ! lsmod | grep "snd_bcm2835" &> /dev/null ; then
  echo 'Loaded sound module'
  modprobe snd_bcm2835
fi

echo 'Exiting Kodi..'
systemctl stop kodi

echo 'Starting moonlight..'
/storage/java/bin/java -jar /storage/moonlight/limelight.jar stream 192.168.0.150

echo 'Finished. Firing Kodi back up..'
systemctl start kodi
EOL
```

6. Run the following command to create the update script for you.
```
cat >/storage/moonlight/update.sh <<EOL
#!/bin/bash

function version { echo "$@" | gawk -F. '{ printf("%03d%03d%03d\n", $1,$2,$3); }'; }

function downloadFile {
    # $1 : first argument must be version
    # $2 : second argument must be file to download
    curl -L -O -s "https://github.com/irtimmer/moonlight-embedded/releases/download/v$1/$2"
}

function updateMoonlight {
	FILE=version
	releases=`curl --silent -L https://github.com/irtimmer/moonlight-embedded/releases/latest`
	current_version=`if [ -f "$FILE" ]; then cat $FILE; fi`
	latest_version=`echo "$releases" | egrep -o "/releases/download/v([0-9]\.*)+/" | egrep -o "v([0-9]\.*)+" | cut -c 2- | head -n 1`

	if [ ! -f "$FILE" ] || [ "$(version "$latest_version")" -gt "$(version "$current_version")" ]; then
		echo "Updating moonlight to $latest_version"
		downloadFile "$latest_version" libopus.so
		downloadFile "$latest_version" limelight.jar
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

7. And finally, make the scripts we just created executable.
```
chmod +x /storage/moonlight/run.sh
chmod +x /storage/moonlight/update.sh
```
8. Now run `/storage/moonlight/update.sh` to download the latest files.

Finished! Whenever you want to play games using moonlight, just run /storage/moonlight/run.sh.
You can even create a link in Kodi to make it as seamless as possible. (Prepend it with systemd-run)
So for example: `system.exec("systemd-run /storage/moonlight/run.sh")`


To update moonlight, just run /storage/moonlight/update.sh

