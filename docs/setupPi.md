# Raspberry Pi Setup & Konfiguration

Diese Anleitung enthält konsolidierte Hinweise zur Einrichtung und Konfiguration eines Raspberry Pi Systems, inkl. Netzwerkinfrastruktur, VNC-Server, Systemdienste, SSH, Java und mehr.

---

## 🔧 SD-Karten-Abbilder erstellen und anpassen

### Image erzeugen (Linux)

```bash
sudo dd bs=1M count=11000 if=/dev/sde of=~/raspberry-pi.img
```

Oder mit Fortschrittsanzeige und Hash:

```bash
sudo apt install dcfldd
sudo dcfldd bs=1M count=11000 if=/dev/sdd of=~/raspberry-pi.img hash=md5
```

### Image verkleinern

- **GParted**: Partition manuell verkleinern.
- **PiShrink**: Automatisches Verkleinern und Komprimieren.  
  [PiShrink Repo](https://github.com/Drewsif/PiShrink/)

```bash
./pishrink.sh raspberry-pi.img
```

> Beim ersten Start expandiert das Image automatisch auf die volle Größe.

---

## 🌐 Netzwerkkonfiguration

### DHCP-Client prüfen (Raspbian Buster+)

```bash
sudo service dhcpcd status
sudo service dhcpcd start
sudo systemctl enable dhcpcd
```

### Statische IP in `/etc/dhcpcd.conf` setzen

```conf
interface eth0
static ip_address=192.168.0.100/24
```

### WLAN als Access Point (mit `nmcli`)

```bash
sudo nmcli d wifi hotspot ifname wlan0 ssid my-ap password s3cretpass
sudo nmcli con mod Hotspot ipv4.method manual ipv4.address 10.11.12.1
sudo nmcli con mod Hotspot connection.autoconnect yes
```

---

## 📡 DNS & DHCP Server (dnsmasq)

```bash
sudo apt install dnsmasq
```

In `/etc/dnsmasq.conf`:

```conf
interface=wlan0
bind-dynamic
domain-needed
bogus-priv
dhcp-range=10.11.12.42,10.11.12.64,255.255.255.0,12h
```

Aktivieren:

```bash
sudo service dnsmasq start
sudo systemctl enable dnsmasq
```

---

## 🖥️ VNC-Server installieren und als systemd-Service einrichten

### Installation

```bash
sudo apt install xfce4 xfce4-goodies tightvncserver
vncserver  # einmalig starten, Passwort festlegen
```

### Datei `~/.vnc/xstartup`:

```bash
#!/bin/bash
xrdb $HOME/.Xresources
startxfce4 &
```

```bash
chmod +x ~/.vnc/xstartup
```

### Service-Datei `/lib/systemd/system/tightvnc-server@.service`

```ini
[Unit]
Description=Remote desktop service (VNC)
After=syslog.target network.target

[Service]
Type=forking
User=pi
Group=pi
WorkingDirectory=/home/pi
PIDFile=/home/pi/.vnc/%H:%i.pid
ExecStartPre=-/usr/bin/vncserver -kill :%i > /dev/null 2>&1
ExecStart=/usr/bin/vncserver -depth 24 -geometry 1920x1080 :%i
ExecStop=/usr/bin/vncserver -kill :%i

[Install]
WantedBy=multi-user.target
```

### Starten:

```bash
sudo systemctl daemon-reload
sudo systemctl start tightvnc-server@2
sudo systemctl enable tightvnc-server@2
```

---

## 🧩 Systemd-Services manuell definieren

Beispiel `/etc/systemd/system/myservice.service`:

```ini
[Unit]
Description=My custom service
After=network.target

[Service]
ExecStart=/path/to/your/script.sh
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable myservice
sudo systemctl start myservice
```

---

## ⏰ Zeit-Synchronisierung (systemd-timesyncd)

```bash
timedatectl status
sudo vi /etc/systemd/timesyncd.conf
```

Beispielkonfiguration:

```ini
[Time]
NTP=ntp1.t-online.de ptbtime1.ptb.de
FallbackNTP=0.debian.pool.ntp.org 1.debian.pool.ntp.org
```

```bash
sudo timedatectl set-ntp true
sudo systemctl restart systemd-timesyncd
```

---

## 🔐 SSH-Zugang und Key-Installation

```bash
sudo apt install openssh-server
ssh-keygen -t ed25519 -C "mein-key"
ssh-copy-id user@192.168.x.x
```

---

## ☕ Java installieren

```bash
sudo apt update
sudo apt install default-jdk
sudo apt install openjdk-8-jdk
sudo update-alternatives --config java
sudo update-alternatives --config javac
```

Optional neuere Version:

```bash
sudo apt install openjdk-19-jdk-headless
```

---

## 🧰 Fehlerbehebung: Abgelaufene GPG-Schlüssel bei apt

```bash
sudo apt-key list
sudo apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 04EE7237B7D453EC 648ACFD622F3D138 DCC9EFBF77E11517
```

---

## 🖥️ GUI-Modus wechseln (headless <-> grafisch)

```bash
# Nur Terminal
sudo systemctl set-default multi-user

# GUI wieder aktivieren
sudo systemctl set-default graphical
```

---

## 🔗 Quellen und Referenzen

- [PiShrink](https://github.com/Drewsif/PiShrink/)
- [Netplan Docs](https://netplan.readthedocs.io/en/latest/)
- [VNC unter Linux](https://linuxhint.com/installing_vnc_server_linux_mint/)
- [Systemd Units](http://manpages.ubuntu.com/manpages/disco/de/man5/systemd.unit.5.html)