# MySQL-Cliente-Servidor
Instalación y Configuración de MySQL en entorno cliente-servidor (Ubuntu 24.04)

La siguiente documentación, explica paso a paso la instalación, configuración y prueba de conexión de MySQL en un servidor y un cliente Ubuntu 24.04 utilizados en isard, así como la posterior creación de una base de datos, tabla e inserción de registros.

Entorno utilizado
Utilizaremos dos máquinas Ubuntu 24.04 con las siguientes interfaces de red:
SERVER: 192.168.1.17
CLIENTE: 192.168.1.16

Configuración inicial del servidor
En este caso empezaremos con una máquina en isard que actuará como servidor que renombraré con:


sudo hostnamectl set-hostname dosorio-server


Instalación de MySQL

Nuestro objetivo es instalar MySQL en el servidor para que actúe como tal, en este caso, he utilizado la siguiente orde:

isard@dosorio-server:~$ sudo apt install mysql-server

Podemos verificar que versión hemos descargado con la orden siguiente:

isard@dosorio-server:~$ mysql -V
mysql  Ver 8.0.43-0ubuntu0.24.04.2 for Linux on x86_64 ((Ubuntu))

Una vez verificada la versión, podemos comprobar su estado y si está funcionando con la siguiente orden:

isard@dosorio-server:~$ sudo systemctl status mysql
● mysql.service - MySQL Community Server
     Loaded: loaded (/usr/lib/systemd/system/mysql.service; enabled; preset: enabled)
     Active: active (running) since Tue 2025-11-18 18:07:05 UTC; 2min 0s ago
    Process: 13340 ExecStartPre=/usr/share/mysql/mysql-systemd-start pre (code=exited, status=0/SUCC>
   Main PID: 13348 (mysqld)
     Status: "Server is operational"
      Tasks: 37 (limit: 7015)
     Memory: 363.9M (peak: 378.2M)
        CPU: 1.884s
     CGroup: /system.slice/mysql.service
             └─13348 /usr/sbin/mysqld

nov 18 18:07:03 dosorio-server systemd[1]: Starting mysql.service - MySQL Community Server...
nov 18 18:07:05 dosorio-server systemd[1]: Started mysql.service - MySQL Community Server.
lines 1-14/14 (END)

Acceso local a MySQL desde el servidor
Para comprobar que podemos entrar a MySQL desde la máquina servidor, entraremos con el siguiente comando:

isard@dosorio-server:~$ sudo mysql
[sudo] password for isard: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 14
Server version: 8.0.43-0ubuntu0.24.04.2 (Ubuntu)

Copyright (c) 2000, 2025, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 

También podemos entrar como root, que es lo más recomendable para trabajar a nivel base en mysql, con el siguiente comando:

mysql -u root –p

Si nos solicita una contraseña y no la sabemos o no la tenemos, podemos crearla con la siguiente orden dentro de mysql:


isard@dosorio-server:~$ sudo mysql
[sudo] password for isard: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 17
Server version: 8.0.43-0ubuntu0.24.04.2 (Ubuntu)

Copyright (c) 2000, 2025, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

Una vez dentro de mysql, usaremos la siguiente orden:



mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Dhayanseña';
Query OK, 0 rows affected (0,02 sec)

Para que se actualicen y se guarden los datos, haremos uso de la siguiente orden:


mysql> flush privileges;
Query OK, 0 rows affected (0,01 sec)

Para la carga de datos, usaremos una base de datos llamada nba, que contiene información relacionada a la misma. Para cargar dicha bbdd, al estar usando un servidor, me he conectado por ssh desde una maquina con ubuntu desktop donde tenia descargado el archivo y con la siguiente orden, lo he copiado a una carpeta dentro del ubuntu servidor:


alumne@dosoriocliente:~/Escriptori/sql$ scp nba.sql isard@192.168.1.17:~/sql
isard@192.168.1.17's password: 
nba.sql                                                            100% 1271KB  35.1MB/s   00:00  


Seguidamente, usaremos las siguientes órdenes dentro de mysql, con las que crearemos la base de datos y los datos que queremos tener en ella, los cargaremos directamente desde el archivo .sql indicando la ruta y el nombre del archivo:



CREATE DATABASE nba;
USE nba;
SOURCE ~/sql/nba.sql
mysql> source ~/sql/nba.sql

Una vez hecho esto, se empezará a cargar la bbdd, lo siguiente será comprobar que la misma contiene las tablas correspondientes y se ha ejecutado correctamente, para ello entraremos con la siguiente orden:

mysql> use nba;
Database changed

El siguiente paso, es comprobar que hayan tablas y que en las tablas hayan valores, para ello mostraremos las tablas y seguidamente, haremos un par de consultas sencillas:


mysql> show tables;
+---------------+
| Tables_in_nba |
+---------------+
| equipos       |
| estadisticas  |
| jugadores     |
| partidos      |
+---------------+
4 rows in set (0,00 sec)

mysql> select * from equipos limit 5;
+-----------+--------------+-------------+-----------+
| Nombre    | Ciudad       | Conferencia | Division  |
+-----------+--------------+-------------+-----------+
| 76ers     | Philadelphia | East        | Atlantic  |
| Bobcats   | Charlotte    | East        | SouthEast |
| Bucks     | Milwaukee    | East        | Central   |
| Bulls     | Chicago      | East        | Central   |
| Cavaliers | Cleveland    | East        | Central   |
+-----------+--------------+-------------+-----------+
5 rows in set (0,01 sec)

Permitir conexiones remotas al servidor MySQL


El siguiente punto, es conseguir que el cliente se pueda conectar al MySQL del servidor a través de la red, para ello, haremos las siguientes dos cosas:

En primer lugar, tendremos que configurar MySQL en el servidor para que escuche la IP de la máquina cliente (no solo en localhost) para ello editaremos el fichero de configuración de MySQL con la siguiente orden:

isard@dosorio-server:~$ sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf

una vez abierto el archivo de configuración, debemos buscar la siguiente línea:
bind-address            = 127.0.0.1 → esto indica que esta escuhando solo al localhost.

La cambiaramos y la dejaremos de la siguiente manera:
bind-address            = 0.0.0.0 → con esto indicamos que escuche a todas las interfaces de red. Lo cual es conveniente si queremos entrar desde la máquina cliente.

Una vez cambiado, guardamos y salimos del archivo. Procedemos a hacer un reinicio del servicio para que se terminen de aplicar los cambios:

isard@dosorio-server:~$ sudo systemctl restart mysql

Y comprobamos su estado, que nos ha de devolver lo siguiente, que indica que está funcionando correctamente:

isard@dosorio-server:~$ sudo systemctl status mysql
● mysql.service - MySQL Community Server
     Loaded: loaded (/usr/lib/systemd/system/mysql.service; enabled; preset: en>
     Active: active (running) since Tue 2025-12-02 02:16:15 UTC; 38s ago
    Process: 1334 ExecStartPre=/usr/share/mysql/mysql-systemd-start pre (code=e>
   Main PID: 1344 (mysqld)
     Status: "Server is operational"
      Tasks: 38 (limit: 7013)
     Memory: 365.5M (peak: 379.9M)
        CPU: 1.422s
     CGroup: /system.slice/mysql.service
             └─1344 /usr/sbin/mysqld

dic 02 02:16:14 dosorio-server systemd[1]: Starting mysql.service - MySQL Commu>
dic 02 02:16:15 dosorio-server systemd[1]: Started mysql.service - MySQL Commun>
lines 1-14/14 (END)


Crear un usuario con permisos de acceso remoto.

El siguiente punto, ya que queremos entrar de manera remota desde una máquina cliente, procederemos a crear un usuario en mysql para que dicho usuario pueda entrar y navegar por la base de datos con los permisos que creamos convenientes, para ello, nuevamente desde la máquina servidor, entramos a MySQL con el usuario root:

isard@dosorio-server:~$ mysql -u root -p

Una vez dentro, creamos un usuario que se pueda conectar desde la red 192.168.1.0 que es en donde están cliente y servidor con la siguiente orden:

mysql> CREATE USER 'dhayancliente'@'192.168.1.%'
    ->   IDENTIFIED BY 'Dhayanseña';
Query OK, 0 rows affected (0,05 sec)

En este caso, le damos permisos totales sobre la base de datos nba con:

mysql> GRANT ALL PRIVILEGES ON nba.* TO 'dhayancliente'@'192.168.1.%';
Query OK, 0 rows affected (0,02 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0,01 sec)

Podemos comprobar que tiene los permisos correspondientes con:

mysql> SHOW GRANTS FOR 'dhayancliente'@'192.168.1.%';
+------------------------------------------------------------------+
| Grants for dhayancliente@192.168.1.%                             |
+------------------------------------------------------------------+
| GRANT USAGE ON *.* TO `dhayancliente`@`192.168.1.%`              |
| GRANT ALL PRIVILEGES ON `nba`.* TO `dhayancliente`@`192.168.1.%` |
+------------------------------------------------------------------+
2 rows in set (0,00 sec)
Efectivamente, así ha sido y ahora nuestro usuario dhayancliente, ya puede entrar en la base de datos nba desde cualquier máquina cliente que esté dentro de la red especificada.

Segunda parte: Cliente 
Ahora es el turno de instalar el cliente MySQL en la máquina cliente para poder establecer la conexión remota, para ello usaremos los siguientes comandos:


alumne@dosoriocliente:~$ sudo apt update

y seguidamente la siguiente orden, la cual no instala el servidor, solo la herramienta mysql para conectarse a un servidor remoto:


alumne@dosoriocliente:~$ sudo apt install mysql-client

Una vez instalado, probamos la conexión al servidor MySQL usando el usuario remoto que hemos creado:

alumne@dosoriocliente:~$ mysql -h 192.168.1.17 -u dhayancliente -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.43-0ubuntu0.24.04.2 (Ubuntu)

Copyright (c) 2000, 2025, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 

Una vez hemos conseguido entrar sin mayor problema, tenemos que comprobar que podemos trabajar con la base de datos de prueba:

mysql> use nba;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+---------------+
| Tables_in_nba |
+---------------+
| equipos       |
| estadisticas  |
| jugadores     |
| partidos      |
+---------------+
4 rows in set (0,00 sec)

mysql> SELECT * FROM jugadores LIMIT 3;
+--------+----------------+-------------+--------+------+----------+---------------+
| codigo | Nombre         | Procedencia | Altura | Peso | Posicion | Nombre_equipo |
+--------+----------------+-------------+--------+------+----------+---------------+
|      1 | Corey Brever   | Florida     | 6-9    |  185 | F-G      | Timberwolves  |
|      2 | Greg Buckner   | Clemson     | 6-4    |  210 | G-F      | Timberwolves  |
|      3 | Michael Doleac | Utah        | 6-11   |  262 | C        | Timberwolves  |
+--------+----------------+-------------+--------+------+----------+---------------+
3 rows in set (0,01 sec)


Como hemos podido comprobar, la configuración y conexión remota, han sido un éxito.


Tercera parte: Cliente remoto (MySQL Workbench)

El siguiente punto a tratar, es la instalación de MySQL Workbench desde la máquina cliente, para ello, lo haremos con snapd ya que es la forma más rápida y sencilla de instalarlo, en primer lugar, instalaremos snapd:

alumne@dosoriocliente:~$ sudo apt install snapd

Una vez instalado snapd, podemos proceder a instalar MySQL Workbench de manera sencilla, ya que esto instalará la versión más reciente de MySQL Workbench sin complicaciones:


alumne@dosoriocliente:~$ sudo snap install mysql-workbench-community

Comprobamos buscando y abriendo el servicio para ver que se ha instalado correctamente:


<figure>
  <img src="./Imágenes/primera.png" >
</figure>

<figure>
  <img src="./Imágenes/segunda.png" >
</figure>

Una vez dentro de MySQL Workbench:
En la pantalla inicial, hacemos clic en "+" o en "New Connection".

<figure>
  <img src="./Imágenes/dieciseis.png" >
</figure>

Rellenamos los datos y le damos a “Test Connection”:

<figure>
  <img src="./Imágenes/tercera.png" >
</figure>

Rellenamos la contraseña de nuestro usuario:

<figure>
  <img src="./Imágenes/cuarta.png" >
</figure>

Si todo ha ido bien, nos debería salir una ventana como la siguiente:

<figure>
  <img src="./Imágenes/quinta.png" >
</figure>

Una vez establecida la conexión, nos saldrá una especie de botón como el siguiente donde esta la base de datos, procederemos a  entrar:

<figure>
  <img src="./Imágenes/sexta.png" >
</figure>

En el panel de la izquierda, seleccionaremos Schemas, seguidamente veremos nuestra base de datos y encima de ella le daremos click derecho y seleccionaremos set as Default Schema:

<figure>
  <img src="./Imágenes/septima.png" >
</figure>

haremos un par de consultas dandole al boton que parece un rayo para ejecutar la query, en la siguiente imagen, se puede comprobar que ha cargado las tablas correctas:

<figure>
  <img src="./Imágenes/octava.png" >
</figure>

Haremos una pequeña consulta para comprobar los valores:

<figure>
  <img src="./Imágenes/novena.png" >
</figure>

Y con esto hemos podido configurar y comprobar el correcto funcionamiento de MySQL Workbench!

Cuarta parte: Cliente (DBeaver)

El siguiente punto, es comprobar que podemos hacer lo mismo utilizando DBeaver desde la máquina cliente, como ya lo tengo instalado desde la práctica de PostgreSQL, iremos directamente a comprobar la conexión. Para ello, abriremos la aplicación de DBeaver y en el apartado de Database, seleccionamos New Database Connection:

<figure>
  <img src="./Imágenes/decima.png" >
</figure>

Elegimos MySQL como tipo de base de datos y le damos a siguiente:

<figure>
  <img src="./Imágenes/once.png" >
</figure>

Seguidamente, rellenamos con los parámetros necesarios para establecer la conexión con nuestra bbdd de nba y le damos a finish:

<figure>
  <img src="./Imágenes/doce.png" >
</figure>

Como se puede comprobar en el Database Navigator, ya aparece nuestra base de datos nba:

<figure>
  <img src="./Imágenes/trece.png" >
</figure>

Si intentamos desplegar la bbdd, automáticamente, nos despliega la siguiente ventana que nos indica que va a descargar los drivers de MySQL, a lo cual aceptaremos para poder trabajar correctamente:

<figure>
  <img src="./Imágenes/catorce.png" >
</figure>

Una vez descargado, vemos y desplegamos el esquema nba y sus tablas. Hacemos click derecho sobre la tabla que queramos y seleccionamos Read data o Edit data y ya nos mostrará nuestra tabla con sus valores:

<figure>
  <img src="./Imágenes/quince.png" >
</figure>

Con esta comprobación, podemos dar por concluída con éxito esta práctica, con todas las configuraciones y conexiones hechas correctamente.


Quinta parte: preguntas


Contesta las siguientes preguntas para relacionar la práctica con la teoría:
¿Consideras MySQL como un SGBD centralizado o bien como uno basado en el sistema cliente/servidor? Argumenta tu respuesta.

 En esta práctica he usado MySQL claramente como un sistema cliente/servidor.
 El servidor MySQL está instalado y centralizado en la máquina server y los clientes (terminal, Workbench y DBeaver) se conectan por red con usuario y contraseña para hacer las consultas.



Si has categorizado MySQL como un sistema cliente/servidor, dentro de qué sistema de capas consideras que encaja mejor: 2 capas o 3 capas? Argumenta tu respuesta.
¿Puedes hacer una comparativa entre esta práctica y la versión de PostgreSQL? ¿Qué similitudes has encontrado? ¿Qué diferencias?

Ambas prácticas son muy parecidas por no decir iguales, en las dos he instalado el servidor en una máquina, he creado usuarios, he configurado el acceso remoto y he probado consultas desde un cliente utilizando distintos métodos.
Las diferencias más notorias están en la configuración, ya que obviamente los ficheros y parámetros son distintos, en los comandos propios de cada servidor de bbdd y en que en PostgreSQL trabajaba con el usuario postgres y el puerto 5432, mientras que en MySQL he usado root y el puerto 3306. Y bueno me he sentido más familiarizado con MySQL ya que es la base de datos que más he trabajado con anterioridad.


webs de referencia:

https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-22-04#step-3-how-to-create-a-dedicated-mysql-user-and-granting-privileges

https://www.hostinger.com/tutorials/how-to-install-mysql-ubuntu?utm_campaign=Generic-Tutorials-DSA-t1|NT:Se|Lang:EN|LO:DE&utm_medium=ppc&gad_source=1&gad_campaignid=16184995375&gclid=EAIaIQobChMIza3kyqT8kAMVi7ODBx1vtRSXEAAYASAAEgK54vD_BwE

https://dev.mysql.com/doc/employee/en/employees-installation.html

https://10web.io/blog/mysql-error-1698/

