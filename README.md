# make_project_nowplaying
# Objectives
* To show on an LCD display, the current played song information Title, Artist, Album, Duration by Volumio (http://www.volumio.org/).
* This Make project is a prototype - building a final solution with a hardware case not planned (yet).

![volumionowplaying-p](https://user-images.githubusercontent.com/47274144/52941292-b6eb4c80-3368-11e9-82fe-24bb5baf4868.png)

![volumionowplaying-a](https://user-images.githubusercontent.com/47274144/52941291-b6eb4c80-3368-11e9-8ede-d3680569ec95.png)

## Functionality
Rasberry Pi NowPlaying Program [B4J Non-UI]:
* Connect to Volumio server and request (in regular intervals, default 10s) song information, via the Music Player Daemon (MPD).
* Display song information on an LCD 20x4 display with the last requested time.
* Autostart and run as a process. Request interval in ms set as parameter.
* Android App showing song information [B4A].
* Windows LCD Display program showing song information in LCD like display [B4J UI].
* Windows MPD Command program to test the Music Player Daemon commands to remote control Volumio [B4J UI].

## Hardware Parts
* 1x Raspberry Pi 3
* 1x LCD 20x4 Display - used a SainSmart IIC/I2C/TWI Serial 2004 20x4 LCD
* 1x Breadboard

## Software
Min software versions required:
* Volumio v.2.041 for the Raspberry Pi
* B4X B4J v4.70 and B4A v6.50

## Credits
LCD display controlled by B4J Library jLCD_I2C which is converted from https://www.b4x.com/android/forum/threads/raspberry-b4j-i2c-hd44780-lcd-driver.61123/ classes (a BIG thanks to the author for developing).

## Wiring
![volumionowplaying_bb](https://user-images.githubusercontent.com/47274144/52941289-b652b600-3368-11e9-88bf-78a0b6c04a6c.png)
```
LCD 2004a = Raspberry Pi (WireColors)
VCC = 5v (red)
GND = GND (black)
SDA = Pin 3 - SDA (blue)
SCL = Pin 5 - SCL (green)
```
## Additional Information
To get started follow the Volumio information.
Download the Volumio Raspberry Pi image from here, create the image, place in the RPi and boot.

### Volumio Hints
Hints collected whilst developing:
* Playlist Location: /data/playlist. The playlist files have JSON format.
* Running Plugin Scripts Example:"node /volumio/app/plugins/system_controller/volumio_command_line_client/commands/setvolume.js 80" to set the volume to 80%.
* Volumio documentation: https://volumio.github.io/docs/ .
* WebSocket API commands https://volumio.github.io/docs/API/WebSocket_APIs.html .
* When issues with recognizing connected USB device, consider to reset Volumio to factory settings.

### Java
To develop & run B4J apps on the Raspberry Pi, Java is required. Java is not part of the Volumio image.
Example installing Java 1.8.0_121 on the Raspberry Pi:
* Download the tar file from the Oracle Website in a folder like /data/javainstall
* cd /data/javainstall
* sudo tar zxvf jdk-8u121-linux-arm32-vfp-hflt.tar.gz -C /opt
* sudo update-alternatives --install /usr/bin/javac javac /opt/jdk1.8.0_121/bin/javac 1
* sudo update-alternatives --install /usr/bin/java java /opt/jdk1.8.0_121/bin/java 1
* sudo update-alternatives --config javac
* sudo update-alternatives --config java
* java -version
* javac -version
* Delete the downloaded tar file

The application has been created for **personal use**  only. If planned for commercial use, ensure to comply to the [Oracle JDK License Agreement](https://www.oracle.com/technetwork/java/javase/terms/license/javase-license.html). 

### B4J Non-UI Application NowPlaying
Note: Recommend to read the various documented source codes for more details.

### Raspberry Pi B4J-Bridge
The B4J application has been developed on a PC and tested using the B4J-Bridge running in a terminal on the Raspberry Pi. In brief:
Open a terminal (using f.e. Putty), create a folder /home/volumio/b4j, download the B4J-Bridge (wget http://www.b4x.com/b4j/files/b4j-bridge.jarhttp://www.b4x.com/b4j/files/b4j-bridge.jar), start sudo java -jar b4j-bridge.jar and connect from the B4J IDE.

### Raspberry Pi NowPlaying.jar
Create a folder /home/volumio/b4j and copy nowplaying.jar to that folder.
For tests, open a terminal and run java -jar /home/volumio/b4j/nowplaying.jar 10000
The parameter 10000 is the request interval, in msecs, for song information from the Volumio server.

### Development Hints
When NowPlaying is running as a process (see next) and changes to the app are made then
* stop the process first
* start the b4j-bridge
* connect from the B4J IDE
* make changes
* run

When finished, copy nowplaying.jar to /home/volumio/b4j folder.

```
cd b4j
ps -ef | grep nowplaying
root 516 1 0 14:39 ? 00:00:00 sudo java -jar /home/volumio/b4j/nowplaying.jar 10000
root 612 516 0 14:39 ? 00:00:15 java -jar /home/volumio/b4j/nowplaying.jar 10000
volumio 1886 1879 0 15:15 pts/0 00:00:00 grep nowplaying
sudo kill 612
sudo java -jar b4j-bridge.jar
B4J-Bridge v1.20
Waiting for connections (port=6790)...
My IP address is: 169.254.nnn.nn
FTP Server started: ftp://169.254.nnn.nn:NNNN
Start B4J-Bridge with -disableftp to disable.
Connected!
```

-Raspberry Pi Autostart NowPlaying
To autostart nowplaying, add start information to rc.local for nowplaying.jar stored in folder /home/volumio/b4j.
Edit rc.local:
```
sudo nano /etc/rc.local
```

Add these lines - Note that nowplaying is started as a process by adding the ampersand & sign:

```
#B4J NowPlaying with request interval 10000ms
sudo java -jar /home/volumio/b4j/nowplaying.jar 10000 &
exit 0
```

Save rc.local followed by a reboot to check if ok.
The LCD display should state NowPlaying About.
Note: The Raspberry Pi Volumio setup does not include crontab, therefor rc.local is used.

### Music Player Deamon (MPD)
The MPD Protocol is used to obtain song information.
Example to request current song information by sending a command sequence:
```
currentsong
close
```
In B4J, the class AsyncstreamsText is used to send strings to the Volumio Server.
Example:
```
Private RequestSongCmd As String = $"currentsong${CRLF}close${CRLF}"$
astreams.Write(RequestSongCmd)
```

Important to write CRLF after each command line and close off with CRLF!

The information returned contains Tags from which Artist, Title, Album, Time are used.
These tags are stored into a map from which the tags are obtained via RequestSongMap.Get("Title") etc.
A small Windows application, written with B4J, to test MPD commands is included (with some example commands).

## Soure Code
The source code and required libraries can be found in archive __nowplaying.zip__. The code is well documented.
