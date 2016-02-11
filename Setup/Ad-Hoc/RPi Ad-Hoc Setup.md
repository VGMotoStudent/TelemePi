# RPi Ad-hoc Network Setup
> El método no está todavia probado y puede no funcionar

## Objetivo
El objetivo es construir un servidor web austosuficiente dentro de una Raspberry Pi con su propia red, a la que solo los ususarios autenticados por contraseña se podrán conectar. Una vez dentro de la red se tendrá acceso al servidor web mediante HHTP o a la propia Raspberry Pi mediante SSH

## Requisitos
 - Raspberry Pi (en este caso modelo B)
 - Adaptador Wi-Fi compatible con Raspberry Pi
 	- *Edimax EW-7811Un Wireless 802.11b/g/n nano USB adapter*
 - Adaptador de corriente USB 5V/1A
 - Cable Ethernet RJ-45
 - Terminal con SSH 
 	- Linux: openssh 
 	- Windows: Putty o similares

## Configuración del sistema
### Instalar linux
Lo primero que debemos hacer es grabar en una SD (o microSD según la el modelo de Raspberry Pi que utilicemos) una imagen de linux para la arquitectura ARM. la más común de todas es Raspbian, basada en Debian y apollada por los creadores de la Raspberry Pi. La version Jessie, la que se utilizará en este caso, se puede descargar de [aquí](https://downloads.raspberrypi.org/raspbian_latest)

Para grabar la imágen se puede usar diferentes metodos, como el comando dd en linux o el programa Win32Diskimager en linux.

Una vez tenemos isntalado linux usamos SSh para conectarnos a la Raspberry Pi.

### Configurar red

#### Conectarse a través del adapatador Wi-Fi
Para conectarnos por primear vez hemos tenido que hacerlo mediante un cable Ethernet conectado a un router. El siguiente paso es configurar la conexión para que se conecte al router pero usando el adaptador inalámbico. Para ello haremos lo siguiente:

1. Obtener el nombvre de la interfaz inalámbrica. Para ello usamos el comando `iwconfig`. Normalmente suele se `wlan0` o `wlan1`.
2. Ejecutar el programa `wpa_cli`, teniendo en cuenta que los datos "miSSID" y "miCONTRASEÑA" han de ser sustituidos por sus respectivos valores. Con el programam haremos lo siguiente:
```
    $ wpa_cli
```
```
> scan
```
```
> scan_results
```
```
> add_network
0
```
```
> set_network 0 ssid "miSSID"
```
```
> set_network 0 psk "miCONTRASEÑA"
```
```
> enable_network 0
```
```
> save_config
```
```
> quit
```
Con esto tendremos configurado la red.

#### Crear opcion para usar red Ad-hoc
En caso de no poder conectarnos al router, haremos que la propia placa cree su proipa red Ad-hoc seguir teniendo la posibilidad de acceder a ella. Para ello haremos lo siguiente.

1. Instalamos el servidor DHCP:
```
sudo apt-get update
sudo apt-get install isc-dhcp-server
```

2. Añadimos la siguiente linea en el fichero ```/etc/default/isc-dhcp-server```. Usando el nombre de nuestra interfaz:
```
INTERFACES="wlan0"
```

3. Añadimos una configuracioón para el servidor DHCP modiicando el archivo ```/etc/dhcp/dhcpd.conf```. la configuración quedaría asi:
```
DHCPDARGS=wlan0;
default-lease-time 600;
max-lease-time 7200;
option subnet-mask 255.255.255.0;
option broadcast-address 10.0.0.255;
option domain-name "RPi-network";
option routers 10.0.0.1; #default gateway
subnet 10.0.0.0 netmask 255.255.255.0 {
	range 10.0.0.2 10.0.0.20; #IP range to offer
}
```

4. Modificamos la configuración de las interfaces para que use primero la interfaz inalámbrica. Esto lo hacemos modificando el archivo ```/etc/network/interfaces```. Quedaría algo como esto:
```
auto lo wlan0
iface lo inet loopback
allow-hotplug eth0
iface eth0 inet dhcp
allow-hotplug wlan0
iface wlan0 inet manual
```

5. Mediante el siguiente comando hacemos que el servidor DHCP no se inicie automáticamente al arranacr el sistema:
```
sudo update-rc.d -f isc-dhcp-server remove
```

6. Creamos un script para que en caso de no poder ocnectarse a la red que hemos configurado, la Raspberry Pi cree su propia red Ad-hoc a la que nos podremos conectar. Para que el script se ejecute nada mas arrancar el dispositivo lo incluiremos en el archivo '''/etc/rc.local'''. El script quedaría de la siguiente manera:

```
# RPi Network Conf Bootstrapper
createAdHocNetwork(){
    echo "Creating ad-hoc network"
    ifconfig wlan0 down
    iwconfig wlan0 mode ad-hoc
    iwconfig wlan0 key aaaaa11111 # la clave de la red
    iwconfig wlan0 essid RPi      # el SSID de la red
    ifconfig wlan0 10.0.0.200 netmask 255.255.255.0 up
    /usr/sbin/dhcpd wlan0
    echo "Ad-hoc network created"
}
echo "================================="
echo "RPi Network Conf Bootstrapper 0.1"
echo "================================="
echo "Scanning for known WiFi networks"
ssids=( 'MyWlan' 'MyOtherWlan' )
connected=false
for ssid in "${ssids[@]}"
do
    if iwlist wlan0 scan | grep $ssid > /dev/null
    then
        echo "First WiFi in range has SSID:" $ssid
        echo "Starting supplicant for WPA/WPA2"
        wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf > /dev/null 2>&1
        echo "Obtaining IP from DHCP"
        if dhclient -1 wlan0
        then
            echo "Connected to WiFi"
            connected=true
            break
        else
            echo "DHCP server did not respond with an IP lease (DHCPOFFER)"
            wpa_cli terminate
            break
        fi
    else
        echo "Not in range, WiFi with SSID:" $ssid
    fi
done
if ! $connected; then
    createAdHocNetwork
fi
exit 0
```

Reiniciamos el sistema y probamos que todo funciona correctamente.
