# Multiple Octoprint instances in Docker

## Setup

As per [sceptre357's awesome post](https://community.octoprint.org/t/setup-multiple-octoprint-instances-in-docker-on-debian-10/31285)

### install docker

```bash
sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/debian/gpg 2 | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian 2 $(lsb_release -cs) stable"
sudo apt-get install docker-ce docker-ce-cli containerd.io 3 docker-compose
```

### Configure UDEV

* Disconnect all printers from your print server except for the one you want to add
* ```ls /dev/serial/by-path/``` copy the path of your printer excluding the “-port0” (should be like pci-0000:00:1a.0-usb-0:1.6:1.0
)
* Create new UDEV rule. UDEV rule MUST begin with a letter or it will prematurely enumerate. ```sudo nano /etc/udev/rules.d/a-usb.rules```
* create a rule for each printer
```SUBSYSTEM=="tty", ENV{ID_PATH}=="pci-0000:00:1a.0-usb-0:1.4:1.0", SYMLINK+="wanhao-d12-0"``` (replace with your printer's path making sure each name is unique "wanhao-d12-0", "ender3-0", ect...)
* Save and reboot

## how to run

Edit the docker-compose.yml file according to your needs

```bash
docker-compose up -d
```

## Add webcam support

The easiest way I found to get webcam support is to run mjpg_streamer on the host machine and adding the url to the octoprint settings (in this case  it would be http://MY-IP:8090/?action=stream): 

```bash
mjpg_streamer -i "/usr/local/lib/mjpg-streamer/input_uvc.so -n -f 10 -r 1280x720" -o "/usr/local/lib/mjpg-streamer/output_http.so -p 8090 -w /usr/local/share/mjpg-streamer/www"
```

I have only tried thois with one camera so far but I assume it is possible with multiple cameras by adding the -d option in the input plugin and assigning a different port to the output plugin.

### Installing mjpg_streamer

I found installation and setup information [here](https://www.sigmdel.ca/michel/ha/rpi/streaming_fr.html)

```bash
sudo apt-get install cmake libjpeg8-dev
wget https://github.com/jacksonliam/mjpg-streamer/archive/master.zip
unzip master.zip
rm -rf master.zip
cd mjp*g-*
cd mjpg-*
make
sudo make install
```