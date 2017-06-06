# Titulo

Un servicio web permite a un proovedor compartir una funcionalidad a través de la red. Un servicio web tiene dos propiedades:
* Es autodescriptivo
* Descubrible

## Creación de servidor virtual

Digital Ocean es un proovedor de servidores virtuales privados (VPS) basados en Linux llamados droplets. Para crear un servidor virtual se siguen los siguientes pasos:

1. Iniciar sesión en Digital Ocean. Seleccionar la opción ```Create Droplet```

![Bloqueo](1.JPG)

2. Seleccionar el SO del droplet y el paquete que más se adecúe a nuestras necesidades.
![Bloqueo](2.JPG)

3. Seleccionar la ubicación del droplet y seleccionar el botón ```Siguente```
![Bloqueo](3.JPG)

4. Esperar mientras termina la creación del droplet.
![Bloqueo](4.JPG)
![Bloqueo](5.JPG)

5. Una vez creado el droplet se recibe una notificación con los datos del droplet que se acaba de crear.
![Bloqueo](6.JPG)

## Apache

Apache es un servidor web HTTP de código abierto para plataformas Unix (BSD, GNU/Linux, etc), Microsoft Windows, Macintoch y otras que implementa el protocolo HTTP/1.11. Es un servidor altamente configurable sin interfaz gráfica. Entre sus principales ventajas se encuentran:
* Modularidad.
* Código abierto.
* Multi-plataforma.
* Extensible
* Popular (Documentación/soporte)

### Instalación y configuración de Apache

* Instalación de paquetes necesarios
```
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get install apache2
```

* Agregar al archivo ``` /etc/apache2/apache2.conf ``` la línea
```
ServerName dominio_del_servidor_o_IP
```

* Se comprueba la configuración de apache para los errores de sintaxis
```
$ sudo apache2ctl configtest
```

* Reiniciar Apache
```
$ sudo systemctl restart apache2
```

## Configuración de host virtual

* Una vez que se tiene instalado Apache se crea la estructura del directorio:
```
$ sudo mkdir -p /var/www/eroland.me/public_html
$ sudo mkdir -p /var/www/clandestina-hds.com/public_html
```

* Cambiar permisos a los directorios anteriores
```
$ sudo chown -R $USER:$USER /var/www/eroland.me/public_html
$sudo chown -R $USER:$USER /var/www/clandestina-hds.com/public_html
```

* Habilitar el acceso de lectura
```
$sudo chmod -R 755 /var/www
```

* Crear una página de prueba para cada sitio y guardarla:

```
/var/www/eroland.me/public_html/index.html
```
El archivo ``` ìndex.html ``` será usado como base para cada uno de los sitios.

* Crear un archivo de configuración ``` 000-default.con ``` para indicar a Apache como gestionar las solicitudes.

```
$ sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/eroland.me.conf
```

* Editar el archivo anterior con permisos de root, debe tener las siguientes directivas:

```
<VirtualHost *:80>
        ServerName eroland.me
        ServerAlias www.eroland.me
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/eroland.me/public_html
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

```ServerName``` Indica cuál es el dominio base que coincidirá con el servidor virtual.

``` ServerAlias ``` Se establecen nombres alternativos del dominio base.

``` ServerAdmin ```  Es donde el administrador del sitio recibirá correos.

``` DocumentRoot ``` Indica la ubicación del archivo raiz dentro del servidor virtual

El archivo de configuración anterior será la base para cada sitio, sólo modificando las directivas anteriormente mencionadas.

* Habilitar los archivos del virtual host.
