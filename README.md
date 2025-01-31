# Pila LEMP de 3/4 Capas - Miguel Cordero González

## Índice

1. [Introducción](#introducción)
2. [Creación de servidores](#creación-de-servidores)
    * [Balanceador](#servidor-balanceador)
    * [Servidor NFS](#servidor-nfs)
    * [Servidores Web](#servidores-web1-y-web2)
    * [Servidor de Base de Datos](#servidor-de-base-de-datos)

---

## Introducción

En este proyecto se implementa una pila LEMP junto con la instalación de WordPress. La estructura consta de:

- **Capa 1:** Servidor Balanceador.
- **Capa 2:** Servidores Web (WEB1 y WEB2), además de una máquina que aloja un servidor NFS y PHP-FPM.
- **Capa 3:** Servidor de Base de Datos.

La base de datos utilizada es **MariaDB**.

Los servidores Web1 y Web2 no están accesibles desde una red pública y utilizarán una carpeta compartida proporcionada por NFS.

---

## Creación de Servidores

### Servidor Balanceador

Para la configuración del balanceador:

- Utilizaremos un aprovisionamiento que permitirá ahorrar tiempo en la configuración. Este proceso incluirá la instalación y configuración de **Nginx**, su arranque y la eliminación de archivos innecesarios.

```bash
sudo apt update
sudo apt install -y nginx
systemctl start nginx
systemctl enable nginx
```

- Una vez iniciada la máquina, accederemos al directorio **/etc/nginx/sites-enabled** y crearemos un archivo llamado "balanceador" con el siguiente contenido:

```nginx
upstream servidoresweb {
    server (direccion IP de servidor web1);
    server (direccion IP de servidor web2);
}

server {
    listen 80;
    server_name balanceador;

    location / {
        proxy_redirect off;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_pass http://servidoresweb;
    }
}
```

![image](Fotos/1.png)
![image](Fotos/2.png)

---

### Servidor NFS

Para la configuración del servidor NFS, utilizaremos el siguiente aprovisionamiento:

```bash
sudo apt update
sudo apt install -y nfs-kernel-server nfs-common php-mysql
```

- Crearemos el siguiente directorio:

![image](Fotos/3.png)

- Luego, editaremos el archivo **/etc/exports** y añadiremos las siguientes líneas para permitir montar las carpetas:

```bash
/var/nfs/compartir/wordpress X.X.X.X(rw,sync,no_root_squash,no_subtree_check)
/var/nfs/compartir/wordpress X.X.X.X(rw,sync,no_root_squash,no_subtree_check)
```

![image](Fotos/4.png)
![image](Fotos/5.png)

- Accedemos a **/var/nfs/compartir/wordpress** y descargamos WordPress:

```bash
sudo wget https://wordpress.org/latest.tar.gz
sudo tar -xzvf latest.tar.gz
```

- Habilitaremos los puertos de los servidores para evitar problemas con las conexiones NFS:

![image](Fotos/8.png)

- Editamos el archivo **wp-config-sample.php** en la carpeta donde descargamos WordPress:

![image](Fotos/10.png)
![image](Fotos/11.png)

- Finalmente, asignamos los permisos adecuados:

![image](Fotos/12.png)

---

### Servidores Web1 y Web2

Para la configuración de los servidores Web1 y Web2, utilizaremos el siguiente aprovisionamiento:

```bash
sudo apt-get update
sudo apt-get install -y nginx nfs-common php-fpm php-mysql
```

Para montar el directorio creado en el servidor NFS:

```bash
sudo mount X.X.X.X:/var/nfs/compartida/wordpress /var/nfs/compartida/wordpress
```

![image](Fotos/13.png)
![image](Fotos/14.png)

- Accedemos al directorio **/etc/nginx/sites-available** y creamos una copia del archivo "default":

![image](Fotos/15.png)

- Modificamos las siguientes líneas:

![image](Fotos/16.png)
![image](Fotos/17.png)

- Una vez editado el archivo, eliminamos el archivo "default" y dejamos el de WordPress:

```bash
sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled
```

![image](Fotos/18.png)

---

### Servidor de Base de Datos

Para la configuración del servidor de base de datos, utilizaremos el siguiente aprovisionamiento:

```bash
sudo apt-get update
sudo apt-get install -y mariadb-server
```

- Una vez arrancada la máquina, accedemos al directorio **/etc/mysql/mariadb** y editamos el archivo "50-server.cnf", modificando la línea "bind-address".

![image](Fotos/19.png)

- Luego, creamos la base de datos, el usuario y asignamos los permisos correspondientes:

![image](Fotos/20.png)
![image](Fotos/21.png)

---

### Comprobación

Para comprobar que la página web está funcionando, abrimos un navegador y accedemos a:

```text
localhost:9000
```

Adicionalmente, instalé una página de base de datos debido a un problema inicial con WordPress, el cual finalmente pude solucionar.

