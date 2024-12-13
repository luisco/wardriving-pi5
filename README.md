# Wardriving con raspberry pi 5

Kismet Version: 2023.07.R1.8


# Componentes

- Raspberry pi 5
- GY-NEO 6M V2
- Antena Gps Externa Conector IPX a SMA 1575Mhz 3-5V 3mt
- Antena Tplink TL-WN722N V1
- Hub Usb
- Velcro
- Powerbank (10000)

# Software

- Raspberry Pi OS (Debian GNU/Linux 12 (bookworm))
- aircrack-ng 
- gpsmon
- kismet (kali)

# Instalación de airmong

sudo apt-get install -y aircrack-ng

# Instalación de gps
```
sudo apt install gpsd gpsd-clients 
```

# Configuración de gpsd

```
sudo raspi-config
```
Interface Options > Serial Port.

Seleccionar 'No' en login shell over serial y  'Yes' en serial port hardware habilitado.

```
sudo nano /etc/default/gpsd
```
```
DEVICES="/dev/ttyAMA0"
GPSD_OPTIONS="-n"
USBAUTO="true"
START_DAEMON="true"
```

Activación del servicio automatico
```
sudo systemctl enable gpsd && sudo systemctl start gpsd
```

# Instalación de kismet

Solo me funciono el kismet para Kali, los demas me sacaban problemas con Libwebsockets

```
wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key --quiet | gpg --dearmor | sudo tee /usr/share/keyrings/kismet-archive-keyring.gpg >/dev/null
echo 'deb [signed-by=/usr/share/keyrings/kismet-archive-keyring.gpg] https://www.kismetwireless.net/repos/apt/release/kali kali main' | sudo tee /etc/apt/sources.list.d/kismet.list >/dev/null
sudo apt update
sudo apt install kismet
```
# Configuración de kismet
```
sudo nano /etc/kismet/kismet.conf
```
Al final
```
gps=gpsd:host=localhost,port=2947,reconnect=true
```


# Script para inicio y ejecución de kismet

```
# Wait for 30 seconds to give the system time to detect and bring up interfaces
sleep 30

# Define the user's home directory and the directory for Kismet data
USER_HOME="/home/pi"
KISMET_DIR="${USER_HOME}/Desktop/wardriving"

# Create the Kismet directory if it doesn't exist
mkdir -p "${KISMET_DIR}"
cd "${KISMET_DIR}"

sudo airmon-ng start wlan1

# Start Kismet with the detected interfaces
KISMET_COMMAND="kismet -c wlan1mon &"


# Function to check if Kismet is running by pinging the web interface
check_kismet() {
    if ! curl -Is http://localhost:2501 | grep -q "200 OK"; then
        $KISMET_COMMAND
    fi
}

# Initial Kismet start
$KISMET_COMMAND

# Loop to check and restart Kismet if not running
while true; do
    sleep 300  # Check every five minutes
    check_kismet
done
```
# Tarea programada para el inicio automatico

```
crontab -e
```

```
@reboot /path/to/your/script.sh
```

# Convertir datos para Wigle

```
kismetdb_to_wiglecsv --in Kismet-20241212-22-53-12-1.kismet --out Kismet-20241212-22-53-12-1.csv
```

# Referencias

- https://gist.github.com/lukeswitz/435be3ff6607a5c8a53c58e2adc4a222
- https://github.com/6vr/Wardriving
