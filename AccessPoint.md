## Configuración de Raspberry Pi como un access point.

Para poder usar la Raspberry Pi como access point se necesita instalar el paquete **hostapd**, también se instalará el paquete **isc-dhcp-server** para configurar un servidor DHCP para el acesss point.
```bash
sudo apt-get update
sudo apt-get install hostapd isc-dhcp-server iptables-persistent
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
### Configuración del servidor DHCP
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
option domain-name "patito.com";
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
sudo update-rc.d isc-dhcp-server enable
```
* Reiniciar Raspberry Pi
