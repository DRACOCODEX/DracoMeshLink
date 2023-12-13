# DracoMeshLink
- Provides a Low Powered Warfighters Mesh Network that uses LoRaWAN and Blockchain to interconnect IoT in the battlefield
- Prototype built w/ HELIUM

## Hardware
+ RAK
  + RAK2287 WisLink LPWAN Concentrator - `https://docs.rakwireless.com/Product-Categories/WisLink/RAK2287`
  + RAK2287 PiHAT - `https://docs.rakwireless.com/Product-Categories/WisHat/RAK2287-Pi-HAT`
  + LoRa and GPS antennas
+ Raspberry Pi 4 Model B - `https://www.raspberrypi.org/products/raspberry-pi-4-model-b`
  + 32 GB Micro SD Card

## Software
+ Balena Etcher - `https://www.balena.io/etcher/`
+ OS X Terminal
+ Raspberry Pi OS 64 Bit Lite, Latest Images: `http://downloads.raspberrypi.org/raspios_lite_arm64/images`
+ RAK2287 Firmware: `https://downloads.rakwireless.com/LoRa/`
+ AWS
+ Docker

## RaspberryPi Configuration
Image your Raspberry Pi microsd card:
+ Download the Raspberry Pi OS 64 bit lite image
` http://downloads.raspberrypi.org/raspios_lite_arm64/images/raspios_lite_arm64-2021-04-09/2021-03-04-raspios-buster-arm64-lite.zip`

+ Flash micro sd card using using Etcher - `https://www.balena.io/etcher/`



+ Set Up Raspberry Pi to be Headless (SSH): `https://www.raspberrypi.org/documentation/configuration/wireless/headless.md`
  + Create new (empty) file on root of `boot` SD card called `ssh` or `ssh.txt`
  + Create `wpa_supplicant.conf` file and place in root of `boot` SD card:
```
country=US
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
network={
  ssid="your_real_wifi_ssid"
  psk="your_real_password"
  }
```

## Pi Basic Config

+ Scan Network for Local Devices to Obtain Raspberry Pi's IP ADDRESS
```console
arp -a
```
+ SSH into raspberry pi @IP Obtained
```console
ssh pi@IP-ADDRESS (Default Password = raspberry)
```

+ Change password
```console
passwd
```

+ Launch Raspbery Pi configurator
```console
sudo raspi-config
```

  + Select `Interface Options`
    + Select `P4 SPI`
    + Select `Yes`
    + Select `Interface Options`
      + Select `P5 I2C`
      + Select `Yes`
  + Select `Interface Options`
    + Select `P6 Serial Port`
    + Select `No` for shell access
    + Select `Yes` for serial port hardware
    + Select `Ok` to confirm
  + Set hostname if desired
  + Save changes and reboot by selecting `Finish`


+ SSH back into raspberry pi

+ Increase the swap file size
    + Turn off swap

    ```console
    sudo dphys-swapfile swapoff
    ```

    + Edit config file
    ```console
    sudo nano /etc/dphys-swapfile
    ```

    + Edit the line 'CONF_SWAPSIZE=100' to 'CONF_SWAPSIZE=1024"
    + Press CTRL-X, and then Y, and then Enter to save changes
    + Reboot with `sudo reboot`

+ SSH back into raspberry pi

+ Update software on the pi
    - Update software repo cache
    ``` console
    sudo apt update
    ```

    - Perform available updates
    ```console
    sudo apt upgrade
    ```

    - Install git and jq
    ```console
    sudo apt install git jq
    ```

+ Install Docker using convience script from https://docs.docker.com/engine/install/debian/

    + Download the script
    ```console
    curl -fsSL https://get.docker.com -o get-docker.sh
    ```

    + Run the install script
    ```console
    sudo sh get-docker.sh
    ```

    + Remove the install script.
    ```console
    rm get-docker.sh
    ```

    + Allow pi user to run docker by adding pi to the docker group.
    ```console
    sudo usermod -aG docker pi
    ```

    + Reboot for changes to take affect.
    ```console
    sudo reboot
    ```

## Set up Miner on Pi
+ SSH back into Raspberry Pi

+ Create directory for miner data
```console
mkdir ~/miner_data
```

+ Get the filename for the most recent version of the miner docker image from quay.io/team-helium/miner.

+ Pull the latest Docker Image and set it to always start up.

```console
sudo docker run -d \
  --env REGION_OVERRIDE=US915 \
  --restart always \
  --publish 1680:1680/udp \
  --publish 44158:44158/tcp \
  --name miner \
  --mount type=bind,source=/home/pi/miner_data,target=/var/data \
  quay.io/team-helium/miner:miner-arm64_2021.05.22.0_GA
```

+ Verify that your container has started
```console
docker ps
```

+ For maximum performance, set up port forwarding on router to the pi.
  + Outside port '44158/TCP'
  + Internal IP / Port 'PI IP ADDRESS:44158'

## Set up Packet Forwarder

+ Make sure your are in your home directory
```console
cd ~
```
+ Clone the packet forwarder for sx1302 based gateways like the RAK2287
```console
git clone https://github.com/helium/sx1302_hal.git
```

+ Move to the project folder
```console
cd sx1302_hal
```

+ Create a ssh key, so we can avoid entering the password a million times in the next step
  + Create the ssh key pair.
    ```console
    ssh-keygen -t rsa
    ```
    Press enter three times to accept the defaults.

    + Set up your user to use the key pair.

    ```console
    ssh-copy-id -i ~/.ssh/id_rsa.pub pi@localhost
    ```
    You will need to enter your password.

+ Compile the project
```console
make clean all
```

+ Make the executables
```console
make install
```

Note you can change parameters for this step in `target.cfg`

+ Install the config files
```console
make install_conf
```

+ Move into bin directory
```console
cd bin
```

+ Make a copy of the conf file for your reggion and name it as the default
```console
cp global_conf.json.sx1250.US915 global_conf.json
```

+ Add pi user to gpio group
```console
sudo usermod -aG gpio pi
```

+ Reboot for changes to take effect
```console
sudo reboot
```

+  SSH back into Raspberry Pi

+ Move to lora packet folder bin directory
```console
cd sx1302_hal/bin
```

+ Test out the packet forwarder
```console
./lora_pkt_fwd
```
  + Test Packet Forwarder and dump to file for Test
  ```console
  ./lora_pkt_fwd > PacketFWD_TEST.txt
  ```
  + Secure Copy from Pi to OS X Local directory (must be logged out of Pi)
  ```console
  scp pi@IP-ADDRESS:PacketFWD_Test.txt .
  ```

+ Stop the packet forwarder by pressing CTRL-c


## Set up auto start scripts

+ Create a systemd service script
```console
sudo nano /etc/systemd/system/lora_pkt_fwd.service
```

+ Paste the following into new text file:

```
[Unit]
Description=LoRa Packet Forwarder
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
WorkingDirectory=/home/pi/sx1302_hal/bin
ExecStart=/home/pi/sx1302_hal/bin/lora_pkt_fwd -c /home/pi/sx1302_hal/bin/global_conf.json
Restart=always
RestartSec=30
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=lora_pkt_fwd
User=pi

[Install]
WantedBy=multi-user.target
```

+ Save the File and Exit

+ Reload the daemon

```console
sudo systemctl daemon-reload
```

+ Enable the service

```console
sudo systemctl enable lora_pkt_fwd.service
```

### The following commands to disable the service, manually start/stop it and check status:

```console
sudo systemctl disable lora_pkt_fwd.service
```

```console
sudo systemctl start lora_pkt_fwd.service
```

```console
sudo systemctl stop lora_pkt_fwd.service
```

```console
sudo systemctl status lora_pkt_fwd.service
```

### Configure rsyslog to redirect the packet forwarder logs into a dedicated file

```console
sudo nano /etc/rsyslog.d/lora_pkt_fwd.conf
```

+ Paste the following into the text editor:
```
if $programname == 'lora_pkt_fwd' then /var/log/lora_pkt_fwd.log
if $programname == 'lora_pkt_fwd' then ~
```

+ Save the File and Exit

+ Restart the rsyslog service

```console
sudo systemctl restart rsyslog
```

+ Reboot for changes to take affect.

```console
sudo reboot
```

### See the Lora Packet Forwarder logs

```console
tail /var/log/lora_pkt_fwd.log
```
## Set up Miner autoupdate script

+ Clone miner auto update script repo
```console
git clone https://github.com/Wheaties466/helium_miner_scripts.git
```

+ CD into directory
```console
cd helium_miner_scripts/
```

+ Make scripte executable
```console
chmod +x miner_latest.sh
```

+ Make the log file
```console
sudo touch /var/log/miner_latest.log
```

+ Run the script
```console
./miner_latest.sh
```

+ Configure cron to run the script daily
```console
sudo crontab -e
```
  + Choose "1" to make nano your editor of choice.

  + Add the following at the end of the text editor.

  ```
  # Check for updates on miner image
  # Use whatever path you have your repo setup with.
  # If you need to test your cron you can use the following site and add "&& curl -s 'https://webhook.site/#!/~'" to the end of your cron and it will make a web request to your specific url when it completes.
  0 */4 * * * /home/pi/helium_miner_scripts/miner_latest.sh >> /var/log/miner_latest.log
  ```
  + Save File

## Wallet
Setup Wallet and Connect to Miner / Gateway.  This is only needed to receive HNT Coins.  Need a HW Device and Key for this.





## Backup and Image Build
Goto Image Build Directory for ReadMe and Scripts
