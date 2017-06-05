## Configuración de Raspberry Pi como un access point.

Para poder usar la Raspberry Pi como access point se necesita instalar el paquete **hostapd**, también se instalará el paquete **isc-dhcp-server** para configurar un servidor DHCP para el acesss point.
```bash
sudo apt-get update
sudo apt-get install hostapd iptables-persistent
```
### Configurar la interface wlan0 con una dirección IP estática.
* Modificar el archivo **/etc/network/interfaces** con las siguientes lineas (comentar la configuración que tenga por defecto)
```bash
iface wlan0 inet static
  address 192.168.23.1
  netmask 255.255.255.0
```
### Configurar la interface wlan0 como access point
* Crear el archivo de configuración para **hostapd**: `sudo nano /etc/hostapd/hostapd.conf` y escribir lo siguiente:
```
interface=wlan0
ssid=raspberry
hw_mode=g
channel=10
macaddr_acl=0
auth_algs=1
wpa=2
wpa_passphrase=raspberry
wpa_key_mgmt=WPA-PSK
wpa_pairwise=CCMP
rsn_pairwise=CCMP
wpa_group_rekey=86400
ieee80211n=1
wme_enabled=1
```
Reemplazar los valores de **ssid** (nombre de la red) y **wpa_passphrase** (contraseña).
* Editar el archivo **/etc/default/hostapd**, encontrar la linea que diga **DAEMON_CONF=** y editarla para que quede de la siguiente manera: **DAEMON_CONF="/etc/hostapd/hostapd.conf"**
* Igualmente editar el archivo **/etc/init.d/hostapd**, encontrar la linea que diga **DAEMON_CONF=** y editarla para que quede de la siguiente manera: **DAEMON_CONF="/etc/hostapd/hostapd.conf"**

### Configurar NAT
Configurar NAT permitirá redireccionar al puerto ethernet las peticiones de los usuarios que se conecten al access point
* Abrir el archivo **/etc/sysctl.conf** y agregar al final:
```
net.ipv4.ip_forward=1
```
* Ejecutar los siguientes comando
```bash
sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
sudo sh -c "iptables-save > /etc/iptables/rules.v4"
```

## Configuración del servidor DHCP

Se desea que cuando un equipo nuevo se conecte a la red este adquiera la configuración IP automáticamente, para ello se instaló un servidor DHCP.

### Servidor DHCP

EL protocolo de configuración dinámica de servidores (DHCP, Dynamic Host Configuration Protocol), es un protocolo de red en el que un servidor provee los parámetros de configuración a las computadoras que lo requieran (máscara de red, puerta de enlace, etc) y también incluye un mecanismo de asignación de direcciones IP.

Un servidor DHCP proporciona opciones de configuración para un equipo cliente. A continuación se muestra una lista de opciones configurables mediante DHCP:

* Dirección del servidor DNS.
* Nombre DNS.
* Puerta de enlace de la dirección IP.
* Dirección de publicación masiva (broadcast-address).
* Máscara de subred.
* Tiempo máximo de espera del ARP (Protocolo de Resolución de Direcciones según siglas en inglés).
* MTU (Unidad de Transferencia Máxima según siglas en inglés).
* Servidores NIS (Servicio de Información de Red según siglas en inglés).
* Dominios NIS.
* Servidores NTP (Protocolo de Tiempo de Red según siglas en inglés).
* Servidores SMTP.
* Servidor TFTP.
* Nombre del servidor WINS.

El servidor DHCP que se instalará es *isc-dhcp-server* ya que es muy sencillo de configurar.

## Instalación y configuración de *isc-dhcp-server*
* Instalar el paquete usando apt
```
sudo apt-get install isc-dhcp-server
```
* Para indicar la interface donde escuchará nuestro servidor se modifica el archivo **/etc/default/isc-dhcp-server**, `INTERFACES="wlan0"`.
* Crear el archivo dhcpd.leases
`touch /var/lib/dhcp/dhcpd.leases`
* Editar el archivo **/etc/dhcp/dhcpd.conf**
```bash
# Reslpaldar el archivo original
sudo cp /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.original
sudo rm /etc/dhcp/dhcpd.conf
sudo nano /etc/dhcp/dhcpd.conf
```
y agregar las siguientes lineas al archivo.
```
subnet 192.168.23.0 netmask 255.255.255.0 {
	range 192.168.23.10 192.168.23.50;
	option broadcast-address 192.168.23.255;
	option routers 192.168.23.1;
	default-lease-time 600;
	max-lease-time 7200;
	option domain-name-servers 8.8.8.8, 8.8.4.4;
}
```
* Finalmente, hacer que los programas se inicien automaticamente.
```bash
sudo systemctl enable hostapd
sudo systemctl isc-dhcp-server enable
```
* Reiniciar Raspberry Pi y conectarse a la red inalámbrica **raspberry**, el servidor DHCP asignará la dirección IP a los clientes que se conecten.
