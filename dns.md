## Servidor DNS

Un servidor de nombres de dominio tiene 2 objetivos fundamentales:
* Traducir una dirección canónica en una dirección IP.
* Traducir una dirección IP en una o varias direcciones canónicas (Traducción inversa).

## Configuración de servidor DNS

* Instalar el paquete **bind9**
```bash
sudo apt-get install bind9
```
* Editar el archivo de configuración: **/etc/bind/named.conf.options**
```
sudo nano /etc/bind/named.conf.options
```
* Descomentar la sección **forwarders** y agregar la dirección IP del dsn del ISP, u otro, como los dns de google (8.8.8.8, 4.4.4.4).
```bash
forwarders {
    8.8.8.8;
 };
```
* Modificar la línea `dnssec-validation auto;` a `dnssec-validation no;`

* Abri el archivo **/etc/dhcp/dhcpd.conf** y en la linea que dice ***option domain-name-servers*** reemplazar la dirección IP por la dirección local del servidor (192.168.23.1).
* Reiniciar los servicios de DHCP y DNS para que se hagan los cambios.
```bash
sudo systemctl restart isc-dhcp-server
sudo systemctl restart bind9
```
### Verificar
* Para verificar que el servidor DNS está funcionando bien, podemos realizar consultas desde un cliente conectado a la red que genera el Raspberry Pi.
* Usando el comando `dig` se puede ver el tiempo que tarda el servidor DNS en hacer la solicitud.
```bash
eroland@debian:~/Descargas$ dig www.google.com | grep 'Query time'
;; Query time: 199 msec
eroland@debian:~/Descargas$ dig www.google.com | grep 'Query time'
;; Query time: 5 msec
eroland@debian:~/Descargas$
```
* Como se puede ver, para la segunda solicitud el tiempo disminuyó, ya que la solicitud solo se hizo al servidor local, el cual ya tenía en cache la dirección IP para el dominio **www.google.com**
