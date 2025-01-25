# Tomcat y Maven



## 1. Instalación de Tomcat
 Instalar OpenJDK

1. Actualizar los repositorios e instalar Nginx:
```bash
   sudo apt update
   sudo apt install nginx
```

### 1. Instalación del Servidor Web Nginx
```bash
    systemctl status nginx
```

## 2. Configuración del Sitio Web

    Crear la estructura de carpetas para el sitio web:

```bash
    sudo mkdir -p /var/www/nginxS/html
```
Clonar el repositorio de ejemplo en la carpeta del sitio web:

```bash
    git clone https://github.com/cloudacademy/static-website-example /var/www/nginxS/html
 ```
Asignar permisos y propietario a los archivos del sitio web:
```bash

    sudo chown -R www-data:www-data /var/www/nginxS/html
    sudo chmod -R 755 /var/www/nginxS
```

Configurar el bloque de servidor para el sitio en Nginx:

```bash
    sudo nano /etc/nginx/sites-available/nginxS
 ```
Consiguraciion de dominio:
 ```bash
    server {
        listen 80;
        listen [::]:80;
        root /var/www/nginxS/html/static-website-example;
        index index.html index.htm index.nginx-debian.html;
        server_name nginxS;
        location / {
            try_files $uri $uri/ =404;
        }
    }

 ```
Crear un enlace simbólico para habilitar el sitio:

 ```bash
    sudo ln -s /etc/nginx/sites-available/nginxS /etc/nginx/sites-enabled/
 ```
Reiniciar Nginx para aplicar los cambios:

```bash
    sudo systemctl restart nginx
```

Hay que tener en cuenta que para que no nos de error nuestro navegador con el servidor de nginx, debemos de ir a:
```bash
    C:\Windows\System32\drivers\etc\hosts 
 ```
y añadir
```bash
    192.168.1.9 nginxS
```
Una vez hecho ya podemos comprobar como se esta acediendo correctamente a los archivos logs en: 
```bash
    /var/log/nginx/access.log
    /var/log/nginx/error.log
 ```

## 3. Configuración de FTP Seguro (FTPS)

Para una transferencia segura de archivos, configuraremos el servidor con FTPS mediante vsftpd.

Instalar el servidor vsftpd:

```bash
    sudo apt-get install vsftpd
```
Crear los certificados SSL para habilitar FTPS:
```bash
    sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/vsftpd.key -out /etc/ssl/certs/vsftpd.crt
```
Si en nuestro Vagrantfile añadiremos este comando para que se den por si solas las respuestas del certificado 
```bash
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/vsftpd.key -out /etc/ssl/certs/vsftpd.crt -subj "/C=ES/ST=./L=./O=./OU=./CN=nginxS/emailAddress=."
```

Configurar vsftpd para utilizar FTPS:

```bash
    sudo nano /etc/vsftpd.conf
```
Actualizar la configuración del vsftpd.conf, en el borraremos estas lineas:

```bash
    rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
    rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
    ssl_enable=NO
```

y las sustituiremos por las siguientes:

 ```bash
    rsa_cert_file=/etc/ssl/certs/vsftpd.crt
    rsa_private_key_file=/etc/ssl/private/vsftpd.key
    ssl_enable=YES
    allow_anon_ssl=NO
    force_local_data_ssl=YES
    force_local_logins_ssl=YES
    ssl_tlsv1=YES
    ssl_sslv2=NO
    ssl_sslv3=NO
    require_ssl_reuse=NO
    ssl_ciphers=HIGH

    local_root=/home/vagrant/ftp
    write_enable=YES
 ```
Tambien seria comveniente descomentar esta linea, para que WinSCP no nos de problemas: 
 ```bash
  write_enable=YES
```
Verificar los permisos de los archivos de certificados y de clave privada:

 ```bash
    ls -l /etc/ssl/certs/vsftpd.crt /etc/ssl/private/vsftpd.key
    sudo chmod 755 /etc/ssl/private
 ```
Reiniciar vsftpd para aplicar los cambios:

 ```bash
    sudo systemctl restart vsftpd
 ```
## 4. Transferencia de Archivos mediante FTPES

Para realizar la transferencia segura de archivos, he obtado por utilizar WinSCP.

Para que me deje acceder por WinSCP, he establecido en el vagrant file una contraseña par el servidor:

 ```bash
    config.vm.provision "shell", name: "set_password", inline: <<-SHELL
    echo "vagrant:caracol" | sudo chpasswd
 ```

Una vez conectados con el servidor a traves de WinSCP, se trasnfieren los archivos necesarios en:
 ```bash
    /home/vagrant/ftp/
 ```
En mi caso, he transferido el index.html y la carpeta assets, que contiene el css, las font, y las img de la web.

Una vez transferidos los archivos, debemos de crear otro provisionamiento para la web de ejemplo que voy a utilizar.

Primero, eliminamos el enlace simbolico que haya en nuestro servidor de cualquier otra web:
 ```bash
    sudo rm /etc/nginx/sites-enabled/nginxS
  ```
Y realizamos los cambios que sean oportunos en todos los archivos para que el nuevo dominio funcione, usando un provisionamiento en el vagrantfile:
 ```bash
  config.vm.provision "shell", name: "paramoreweb", run: "never", inline: <<-SHELL
    sudo rm /etc/nginx/sites-enabled/nginxS
    sudo mkdir -p /var/www/paramoreweb/html
    sudo cp -r /home/vagrant/ftp/* /var/www/paramoreweb/html
    sudo chown -R www-data:www-data /var/www/paramoreweb/html
    sudo chmod -R 755 /var/www/paramoreweb
    cp -v /vagrant/paramoreweb /etc/nginx/sites-available/
    sudo ln -s /etc/nginx/sites-available/paramoreweb /etc/nginx/sites-enabled/
    sudo systemctl restart nginx
```

Creo el nuevo archivo paramoreweb que ira destinado a 
```bash
    /etc/nginx/sites-available/
```
El cual contiene lo siguiente:
```bash 
  server {
	listen 80;
	listen [::]:80;
	root /var/www/paramoreweb/html/static-website-example;
	index index.html index.htm index.nginx-debian.html;
	server_name paramoreweb;
	location / {
		try_files $uri $uri/ =404;
	}
}

```
Vuelvo a modificar el archivo hosts de mi Windows, para que asocie la IP a paramoreweb:

```bash
   C:\Windows\System32\drivers\etc\hosts
```
modificamos el nombre nginxS por paramoreweb
```bash
   192.168.1.9 paramoreweb
```

Y ya tendramos lista nuestra Web.

<<<<<<< HEAD
Aqui muestro las capturas de patalla para visualizar la transferencia de datos (index y asset) de local a la maquina virtual.

<img src="img/1.png"/>
<img src="img/2.png"/>

Y aqui podemos visualizar nuestra web:

<img src="img/3.png"/>

#Cuestiones finales 
=======
## 5. Cuestiones finales
>>>>>>> b6276b2fb94e5fe09707cd6e037fba8fff9f4b27

¿Qué sucede si no realizo el enlace simbólico entre sites-available y sites-enabled para mi sitio en Nginx?

Sin este enlace simbólico, Nginx no puede reconocer ni cargar la configuración del sitio porque solo se fija en las configuraciones dentro de sites-enabled. Al no vincular el archivo de configuración, el sitio no estará disponible y cualquier intento de acceder a él generará un error o redirigirá a la página de configuración predeterminada. Es decir, Nginx actuará como si esa configuración específica no existiera.

¿Qué ocurre si no asigno los permisos correctos a la carpeta /var/www/nombre_web?

Sin los permisos adecuados, Nginx no podrá leer o acceder a los archivos necesarios para mostrar el contenido del sitio. Es importante que el usuario www-data, bajo el cual opera Nginx, tenga permisos de lectura en los archivos y de ejecución en los directorios. Sin estos permisos, el acceso al sitio dará errores. Para evitar problemas, los directorios suelen configurarse con permisos 755, lo cual permite tanto el acceso como la navegación entre carpetas.



# Autentificación


### 1. Comprobación de OpenSSL: Primero, verificamos que el paquete de herramientas de OpenSSL esté instalado en el sistema. Para ello, usamos el siguiente comando:
```bash
   dpkg -l | grep openssl

```
### 2.Creación del archivo .htpasswd: A continuación, creamos el archivo .htpasswd en la ruta /etc/nginx para almacenar los usuarios y contraseñas. Primero, añadimos el nombre de usuario:

```bash
   sudo sh -c "echo -n 'nombre:' >> /etc/nginx/.htpasswd"

```
Luego, generamos la contraseña encriptada y la añadimos al archivo:

```bash
  sudo sh -c "openssl passwd -apr1 >> /etc/nginx/.htpasswd"

```
podemos verificar que el archivo .htpasswd se ha creado correctamente con el siguiente comando:
```bash
   cat /etc/nginx/.htpasswd

```
###  3. Configuración del bloque del servidor en Nginx: Ahora, editamos el archivo de configuración del sitio web en Nginx para aplicar las restricciones de autenticación. Para ello, abrimos el archivo correspondiente:
```bash
sudo nano /etc/nginx/sites-available/paramore


```
Dentro del archivo, agregamos las directivas de autenticación al bloque location de la siguiente manera:
```bash

server {
    listen 80;
    listen [::]:80;
    root /var/www/paramore/html;
    index index.html index.htm index.nginx-debian.html;
    server_name perfect_learn;

    location / {
        auth_basic "Área restringida";
        auth_basic_user_file /etc/nginx/.htpasswd;
        try_files $uri $uri/ =404;
    }
}

```

### 4. Reinicio del servicio Nginx: Para aplicar los cambios realizados en la configuración, reiniciamos el servicio Nginx con el siguiente comando:
```bash
sudo systemctl restart nginx

```
Captura de la creacion de contraseñas a traves del comando 
```bash
sudo sh -c "openssl passwd -apr1 >> /etc/nginx/.htpasswd"

```
<img src="capturas/1.png"/>

## Tarea 2.1. - Ficheros error.log y access.log

Captura del intento de acceder a la web, donde pide autentificación, la introduzco correctamente y entro:

<img src="capturas/2.png"/>
<img src="capturas/3.png"/>
<img src="capturas/4.png"/>

Captura del intento de acceser a la web con credenciales no autorizadas:
<img src="capturas/6.png"/>

Capturas de los sucesos y registros en los logs access.log y error.log:
error.log
<img src="capturas/7_error.png"/>
access.log
<img src="capturas/8_validos.png"/>


## Tarea 2.2. - Acceso restringido a la sección Contact
Se vuelve a editar el archivo paramore de /etc/nginx/sites-available, y esta vez añadiemos la localización de la sección contact para la autenticación.
```bash

server {
	listen 80;
	listen [::]:80;
	root /var/www/paramore/html/perfect_learn;
	index index.html index.htm index.nginx-debian.html;
	server_name perfect_learn;
	location = /contact.html {
		deny 192.168.57.1;
		allow all;
		try_files $uri $uri/ =404;
	}
}

```
<img src="capturas/9_contact.png"/>
<img src="capturas/10_contact.png"/>


## Tarea 3.2. - Doble autenticación, IP y usuario

Se configura Nginx para que, desde la máquina anfitriona, se requiera tanto una IP válida como un usuario autorizado para poder acceder.

Para ello, se modifica el bloque de servidor (server block) de la siguiente manera, incorporando las restricciones de IP y autenticación de usuario al mismo tiempo:
```bash
server {
    listen 80;
    listen [::]:80;
    root /var/www/paramore/html/perfect_learn;
    index index.html index.htm index.nginx-debian.html;
    server_name perfect_learn;

    location privado/privado.html {
        deny 192.168.57.1;
        deny all;
        try_files $uri $uri/ =404;
    }
}
```
<img src="capturas/12_permisoDenegado.png"/>

En esta parte no puedo añadir captura ya que me era imposible dar con la ip correcta que debo de poner en el location, intenté varias opciones pero todas me dejaban entrar sin problema a privado.html

### Documentos paramore.

Adjunto en el proyecto, 5 tipos de archivos llamados paramore que serían los que se van cambiando en sudo nano /etc/nginx/sites-available/paramore para realizar las distintas tareas de la practica

## NOTA IMPORTANTE

En este repositorio el proyecto cuenta con pocos commit debido a que en el ultimo momento no me hacian los push correctamente, por lo que he tenido que crear uno de nuevo con todo el contenido del repositorio anterior. 

## Configuración de Acceso Seguro

### 1. Creación del Directorio y Copia del Sitio Web
Primero, se crea el directorio donde se alojará el sitio web example.com, y luego se copia el contenido del sitio desde la carpeta F
sudo mkdir -p /var/www/example.com/html
```bash
sudo mkdir -p /var/www/example.com/html
sudo cp -r /home/vagrant/ftp/example/* /var/www/example.com/html
```
Luego, se crea el archivo de configuración para example.com:
```bash
sudo nano /etc/nginx/sites-available/example.com
```
En este archivo, se define el directorio raíz y el nombre del servidor para el sitio example.com:
```bash
server {
    listen 80;
    listen [::]:80;
    root /var/www/example/html;
    index index.html index.htm index.nginx-debian.html;
    server_name example.com www.example.com;
    location / {
        try_files $uri $uri/ =404;
    }
}
```
A continuación, se comprueba que no haya errores de sintaxis en la configuración de Nginx:
```bash
sudo nginx -t
```
Y luego se recarga el servicio de Nginx para aplicar los cambios:
```bash
sudo systemctl reload nginx
```

### 2. Configuración del Cortafuegos (UFW)
Para gestionar el cortafuegos, se utilizará UFW (Uncomplicated Firewall). Primero, instalamos UFW si no está instalado:
```bash
sudo ufw enable
sudo ufw status verbose
sudo ufw allow 'Nginx Full'
sudo ufw allow 'Nginx HTTP'
```
verificamos las reglas de UFW con:
```bash
sudo ufw status

```

Y el resultado obtenido es:
```bash
Status: active

To                         Action      From
--                         ------      ----
Nginx Full                 ALLOW       Anywhere
Nginx Full (v6)            ALLOW       Anywhere (v6)

```
Reiniciamos Nginx
```bash
sudo ufw reload
```
<img src="capturas/01.png"/>
### 3. Generación de un Certificado SSL Autofirmado
Para habilitar HTTPS en el servidor, se genera un certificado SSL autofirmado utilizando el siguiente comando de OpenSSL:
```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/example.com.key -out /etc/ssl/certs/example.com.crt
```
<img src="capturas/02.png"/>

### 4. Configuración del Servidor Web con SSL
A continuación, se debe modificar el archivo de configuración de Nginx para habilitar HTTPS, utilizando el certificado SSL recién creado. Se edita el archivo /etc/nginx/sites-available/example.com y se realiza la configuración:
```bash
server {
    listen 80;
    listen 443 ssl;
    root /var/www/example/html/example;
    index index.html index.htm index.nginx-debian.html;
    server_name example.com www.example.com;
    ssl_certificate /etc/ssl/certs/example.com.crt;
    ssl_certificate_key /etc/ssl/private/example.com.key;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    location / {
        try_files $uri $uri/ =404;
    }
}


```
<img src="capturas/03.png"/>
Con la configuración en su lugar, se comprueba la validez de la configuración de Nginx:
```bash
sudo nginx -t
```
Luego, se recarga Nginx para que los cambios surtan efecto:
```bash
sudo systemctl reload nginx
```

### 5. Comprobación de la Configuración
Para comprobar que el sitio web esté accesible, se debe modificar el archivo hosts de la máquina local para asociar la dirección IP de la máquina virtual con el dominio example.com.

En sistemas Windows, el archivo hosts se encuentra en:
```bash
C:\Windows\System32\drivers\etc\hosts
```
Se edita el archivo y se agrega la siguiente línea, donde 192.168.1.9 es la dirección IP de la máquina virtual:
```bash
192.168.1.9 example.com
```
<img src="capturas/04.png"/>
