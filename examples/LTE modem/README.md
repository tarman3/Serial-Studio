# Serial Studio and LTE modem HUAWEI K5161H

## Overview

This project demonstrates how to use **Serial Studio** to visualize signal quality data from a **LTE modem HUAWEI K5161H**.

![LTE modem signal](doc/screenshot.png)

These examples are implemented in OS [Archlinux](https://archlinux.org/).

In Archlinux **Serial Studio** can be
- run from [AppImage](https://github.com/serial-studio/serial-studio/releases/latest)
- installed from AUR [manually](https://wiki.archlinux.org/title/Arch_User_Repository#Installing_and_upgrading_packages) [serial-studio-bin](https://aur.archlinux.org/packages/serial-studio-bin)
- installed by AUR helper, e.g. `yay` 

```
yay -S serial-studio-bin
```

Python was used to receive, process and generate data frames.
Data from HUAWEI K5161H can be get by url API `http://192.168.9.1/api/device/signal`

---

## Method 1 - Virtual Serial Port

### Create Virtual Serial Port
1. Install [socat](http://www.dest-unreach.org/socat/)

```
sudo pacman -S socat
```

2. Create virtual serial port [*](https://www.baeldung.com/linux/make-virtual-serial-port)
```
socat -d -d pty,rawer,echo=0,link=/tmp/ttyV0,baud=9600 pty,rawer,echo=0,link=/tmp/ttyV1,baud=9600
```

3. You can check the operation of the ports by reading the file `/tmp/ttyV0`
```
cat /tmp/ttyV0
```
and writing data to the file `/tmp/ttyV1`
```
echo '100' > /tmp/ttyV1
```

5. Install `python-pyserial`
```
sudo pacman -S python-pyserial
```

6. Run a Python script `lte_serial.py` to process data and send it to the serial port
```
python lte_serial.py
```

### Serial Studio configuration for virtual serial port
- Run **Serial Studio**
- Select FRAME PARSING - Parse via JSON Project File
- Select Project file: `lte.json`
- Select DEVICE SETUP - I/O Interface: Serial Port
- Manually enter COM Port - /tmp/ttyV0
- Click **Connect**  

After get first frame of data Serial-Studio will automatic open dashboard with plots.  

---

## Method 2 - MQTT

### Prepare MQTT broker
1. Install MQTT broker [Mosquitto](https://mosquitto.org/)
```
sudo pacman -S mosquitto
```

2. Run MQTT broker with default settings
```
mosquitto --verbose
```

3. You can check MQTT broker by sending data
```
mosquitto_pub -m "abcd,100,50,75,89" -t "lte"
```

4. Install Python client library for MQTT - **paho**
```
sudo pacman -S python-paho-mqtt
```

5. Run a Python script `lte_mqtt.py` to process data and send it to the MQTT broker
```
python lte_mqtt.py
```

### Serial Studio Configuration for MQTT

- Run **Serial Studio**
- Select FRAME PARSING - Parse via JSON Project File
- Select Project file: `lte.json`
- Click icon MQTT in the top bar
- Set the **Host**: 127.0.0.1
- Set the **Port**: 1883
- Set the **Topic**: lte
- Set the **Mode**: Subscriber
- Set the **Keep Alive**: 600
- Click **Connect**  

![Serial Studio Quick Plot](doc/mqtt_setup.png)

After get first frame of data Serial-Studio will automatic open dashboard with plots.  

---

## Method 3 - UDP Socket
Solution with UDP Socket looks much simpler than other.

- Run a Python script `lte_udp.py` to process data and send it to the UDP Socket
```
python lte_udp.py
```

### Serial Studio Configuration for UDP Socket

   - Run **Serial Studio**
   - Select FRAME PARSING - Parse via JSON Project File
   - Select Project file: `lte.json`
   - Set the **DEVICE SETUP**: I/O Interface: Network Socket
   - Set the **Socket type**: UDP
   - Set the **Remote address**: 127.0.0.1
   - Set the **Local port**: 5005
   - Click **Connect**  
![Serial Studio Quick Plot](doc/udp.png)

After get first frame of data Serial-Studio will automatic open dashboard with plots.  
