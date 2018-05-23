# Video Streaming from RPi

Tutorial copied from: https://docs.emlid.com/navio/common/dev/video-streaming/

First things first. You need to expand filesystem and enable camera using Raspberry Pi configuration tool. Type the following command in console:
```script
pi@dronex: ~ $ sudo raspi-config
```
Raspi-config menu will appear on your screen. After changing the options press the Finish button. Expand filesystem and enable camera options require a reboot to take effect. Raspi-config will ask if you wish to reboot now when you select the Finish button.

Now install dependancies:
```script
pi@dronex: ~ $ sudo apt-get install gstreamer1.0
```

Start streaming:
```script
pi@dronex: ~ $ raspivid -n -w 1280 -h 720 -b 1000000 -fps 15 -t 0 -o - | gst-launch-1.0 -v fdsrc ! h264parse ! rtph264pay config-interval=10 pt=96 ! udpsink host=<remote_ip> port=5600
```
where <remote_ip> is the IP of the device you're streaming to.

Adjust bitrate with -b switch or -fps if your video lags behind.

## Autostarting on boot
To automatically start videostreaming on boot you need to create systemd service:
```script
sudo touch /etc/systemd/system/raspicam.service
```
Edit the service to make it look like this:
```script
[Unit]
Description=raspivid
After=network.target

[Service]
ExecStart=/bin/sh -c "/usr/bin/raspivid -rot 180 -n -w 1280 -h 720 -b 1000000 -fps 15 -t 0 -o - | gst-launch-1.0 -v fdsrc ! h264parse ! rtph264pay config-interval=10 pt=96 ! udpsink host=<remote_ip> port=5600"

[Install]
WantedBy=default.target
```
Don't forget to set IP of the device you're streaming to.

After that run these commands:
- to let systemd know of this service
```script
sudo systemctl daemon-reload
```
- to enable on boot:
```script
sudo systemctl enable raspicam
```
- to test it out
```script
sudo systemctl start raspicam
```
From now on raspivid will start automatically on boot.
To disable autostart run this command:
```script
sudo systemctl disable raspicam
```

Other useful tutorial:
https://dev.px4.io/en/qgc/video_streaming_wifi_broadcast.html

