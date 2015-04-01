# LimeRPi2-kodi
Limelight on OpenELEC. (Kodi)

Uses [the limelight project](https://github.com/irtimmer/limelight-embedded).

Installation:
--------------
1. Put [Java for ARM](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-arm-downloads-2187472.html) on the pi in /storage/java and test it. Fir
```
mkdir -p /storage/java
...extract the java archive and put the contents of the java_1. folder in the /storage/java folder...
/storage/java/bin/java -version
```

2. Put the limelight and libopus files in /storage/limelight.
```
mkdir -p /storage/limelight
cd /storage/limelight
curl -o libopus.so https://github.com/irtimmer/limelight-embedded/releases/download/v1.2.2/libopus.so
curl -o limelight.jar https://github.com/irtimmer/limelight-embedded/releases/download/v1.2.2/limelight.jar
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

if lsmod | grep "snd_bcm2835" &> /dev/null ; then
  echo 'Loaded sound module'
  modprobe snd_bcm2835
fi

echo 'Exiting Kodi..'
systemctl stop kodi

echo 'Starting limelight..'
/storage/jdk/bin/java -jar /storage/limelight/limelight.jar stream 192.168.0.150

echo 'Finished. Firing Kodi back up..'
systemctl start kodi
EOL
```

5. And finally, make the script we just created executable.
```
chmod +x /storage/limelight/run.sh
```


Finished! Whenever you want to play games using limelight, just run /storage/limelight/run.sh.
You can even create a link in Kodi to make it as seamless as possible.

