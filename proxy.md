# Proxy cache
### Configuración de proxy
* Instalar squid3
```
sudo apt-get install squid3
```
* La configuración se realiza en el archivo **/etc/squid3/squi3.conf**, abrir el archivo y buscar la linea que dice ***acl CONNECT method CONNECT***. Abajo de esa linea se define la ACL para nuestra red.
```bash
#ACL para nuestra red
acl mired src 192.168.23.0/255.255.255.0
```
* Buscar la sección que empieza con ***INSERT YOUR OWN RULE(S) HERE TO ALLOW ACCESS FROM YOUR CLIENTS*** y ahí permitir nuestra ACL.
```bash
#Permitimos el accesso a esa ACL
http_access allow mired
```
* Guardamos y reiniciamos el servicio de **squid3**
```bash
/etc/init.d/squid3 restart
```
* Para probar debemos configurar el navegador de un cliente para que se conecte usando nuestro proxy. En firefox podemos configurar el proxy en la sección de **Connection Settings**, ahí escribimos la dirección IP de nuestro servidor en el campo de HTTP Proxy.
![Image of Yaktocat](proxy1.png)

### Proxy transparente
* Ya se ha configurado el servidor proxy, sim embargo, es tedioso tener que configurar todas las computadoras de los clientes. La solución a este problema es configurar el proxy de forma transparente.
* Abrimos el archivo de configuración y buscamos la linea ***http_port 3128*** y agregamos la palabra **transparent**
```bash
http_port 3128 transparent
```
* Instalar el paquete iptables para redirigir el trafico al puerto de squid.
```bash
sudo apt-get install iptables
```
* El siguiente comando redirige todas las peticiones del puerto 80 al puerto 3128 (squid3).
```bash
iptables -t nat -A PREROUTING -p tcp -s 192.168.23.0/24 --dport 80 -j DNAT --to 192.168.23.1:3128
```
