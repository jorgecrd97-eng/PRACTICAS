# Tecnológico Nacional de México
## Instituto Tecnológico de Milpa Alta II
### SISTEMAS COMPUTACIONALES

#### Fundamentos Ingenieria de Software
#### Profesor Diego Alberto Bollas Paredes 

##### alumno Cruz Duran Jorge Isaac
##### NC: 241190049
---


# Migracion de una base de Datos

## Introduccion

## Creacion de la base de datos

empezaremos creando la base de datos y la usaremos para poder hacer nuestras tablas y rellenarlas
``` sql
CREATE DATABASE nombre;
USE nombre;
```
![imagen1](img/Screenshot%20From%202026-11-25%2011-04-16.png)


hacemos las tablas 

```sql
CREATE TABLE nombre_tabla(
);
```
![imagen2](img/Screenshot%20From%202026-11-25%2011-05-01.png)

rellenamos las tablas
```sql
INSERT INTO n_tabla (campos) VALUES(campos);
```
![imagen3](img/Screenshot%20From%202026-11-25%2011-05-32.png)

Creamos la vista
```sql
CREATE VIEW nombre_de_la_vista AS
SELECT columna1, columna2, columna3
FROM nombre_tabla1
JOIN nombre_tabla2
ON condicion_de_relacion;
```
![imagen2](img/Screenshot%20From%202026-11-25%2011-06-20.png)

---

## Registrar los datos de origen 

```
sudo mariadb -e "
SELECT COUNT(*) establece cuánta información puede perder una empresa como máximo ante un problema.
- RTO: Es el objetivo de tiempo de recuperación e indica cuánto tiempo puede permanecer detenido un sistema antes de volver a funcionar.AS autores FROM biblioteca.autores;
SELECT COUNT(*) AS libros FROM biblioteca.libros;
CHECKSUM TABLE biblioteca.autores, biblioteca.libros;"
```

En esta parte verificaremos la cantidad de registros existentes en la tabla de autores y libros, además de la suma CHECKSUM de ambas tablas.

Se utiliza para revisar que la información en la base de datos sea correcta y conocer el estado.
![imagen5](img/Screenshot%20From%202026-11-25%2011-06-52.png)

## respaldo de seguridad
``` 
sudo mariadb-dump --databases biblioteca > respaldo_previo.sql
```

creamos un respaldo de la base de datos biblioteca guardando toda la estructura en respaldo_previo.sql

maria-dump es una herramienta que nos permite generar copias de segurida

--databases realizará el respaldo de una o varias bases de datos, incluyendo las instrucciones para crearlas.

(>) redirige a un archivo

comprobacion de que se creo el respaldo de seguridad

```
ls -lh respaldo_previo.sql
```
nos arrojara el archivo creado
![imagen6](img/Screenshot%20From%202026-11-25%2011-09-44.png)

---

## Exportar la base de datos 5

```
sudo mariadb-dump --single-transaction --routines --triggers --events biblioteca > biblioteca_migracion.sql
```
en esta parte  
--single-transaction Realiza el respaldo dentro de una sola transacción para obtener una copia consistente de los datos
--routines Incluye los procedimientos almacenados y las funciones de la base de datos.

--triggers Incluye los disparadores (TRIGGERS) asociados a las tablas.

--events Incluye los eventos programados de la base de datos.

Este comando permite realizar un respaldo completo de la base de datos biblioteca, ya que no solamente guarda las tablas y sus registros, sino que también incluye procedimientos almacenados, funciones, disparadores y eventos programados.


![imagen7](img/Screenshot%20From%202026-11-25%2011-14-23.png)

## Crear la base de datos de destino

```
sudo mariadb -e "CREATE DATABASE biblioteca_nueva;"
```
en este comando creamos la nueva base de datos que se llamara biblioteca_nueva

![imagen8](img/Screenshot%20From%202026-11-26%2015-48-16.png)

comprobacion de que fue creada
```
sudo mariadb -e "SHOW DATABASES;"
```
observamos que si esta la biblioteca_nueva

![imagen9](img/Screenshot%20From%202026-11-26%2015-48-28.png)

---

## importar en el destino

```
sudo mariadb biblioteca_nueva < biblioteca_migracion.sql
```
en este comando nos ayudara a restaurar la información contenida en el archivo biblioteca_migracion.sql dentro de la base de datos biblioteca_nueva.

![imagen10](img/Screenshot%20From%202026-11-26%2015-49-19.png)

---

## validar la migración
### comparar conteo y checksum

```
sudo mariadb -e "
SELECT COUNT(*) AS autores FROM biblioteca_nueva.autores;
SELECT COUNT(*) AS libros FROM biblioteca_nueva.libros;
CHECKSUM TABLE biblioteca_nueva.autores, biblioteca_nueva.libros;"
```

Se utiliza principalmente para comprobar que la información de la base de datos se haya importado correctamente después de realizar una migración o restauración.
veremos nuevamente nuestros autores y los libros que teniamos en nuestra base de datos desde la biblioteca_nueva

![imagen12](img/Screenshot%20From%202026-11-26%2015-51-17.png)


### verificar que la vista funciona

```
sudo mariadb -e "SELECT * FROM biblioteca_nueva.v_libros_autor;"
```
este comando es una vista que nos permitira consultar y mostrar los registros y columnas de v_libros_autor que se encuentran dentro de biblioteca_nueva

![imagen13](img/Screenshot%20From%202026-11-26%2015-55-41.png)

### verificar la integridad referencial

```
sudo mariadb -e "INSERT INTO biblioteca_nueva.libros (titulo, anio, autor_id) VALUES ('Prueba', 2020, 999);"
```
![imagen14](img/Screenshot%20From%202026-11-26%2016-48-56.png)

### verificar la estructura

```
sudo mariadb -e "SHOW CREATE TABLE biblioteca_nueva.libros\G"
```
![imagen15](img/Screenshot%20From%202026-11-26%2016-50-07.png)



---
## Resultado
|comprobación   |origen|destino|
|---------------|----|---|
|Fila autores   |   3|3 |
|Fila en libros |4   |4 |
|checksum autores| 4192191419  |4192191419  |
|checksum libros|1759993422    | 1759993422 |
|     vista libro     |   si |si   |
|    clave forane    |   si | si  |



## Preguntas 
1. Por qué se hace un respaldo antes de migrar?
por si tenemos un error durante la migracion de la base de datos poder tener la seguridad que no perderemos una base de datos importante


2. ¿Para qué sirve la opción --single-transaction?

 para realizar una copia consistente de los datos en una sola transacción

3. ¿Qué diferencia hay entre exportar con y sin --databases?

    Con --databases el archivo incluye instrucciones para crear y seleccionar la base de datos original.
    Sin --databases exporta únicamente la estructura y los datos, permitiendo importarlos en una base de datos con otro nombre.

4. ¿Qué dos pruebas permiten confirmar que los datos migraron sin pérdidas?

    Comparar la cantidad de registros de las tablas de origen y destino.
    Comparar los valores de CHECKSUM TABLE para comprobar que los datos coinciden.

5. Si el destino estuviera en otro servidor, ¿qué paso adicional sería necesario?

Sería necesario copiar el archivo al otro servidor y después importarlo en la base de datos de destino utilizando MariaDB


## conclusión

La migración de bases de datos nos puede ayudar a cambiar de programa o actualizar nuestros sistemas y modernizar los programas de una empresa sin perder la información almacenada. También nos permite trasladar nuestras bases de datos a nuevos servidores o a la nube, facilitando el acceso y la administración de la información. Por eso es importante realizar respaldos y verificar que todos los datos se hayan migrado correctamente para evitar pérdidas de información.

## anexo

- SCP (Secure Copy) Es un comando que permite copiar archivos de un servidor a otro de forma segura mediante una conexión SSH.
- CSV Es un formato de archivo que permite almacenar información en forma de tablas, separando los datos por comas. Se utiliza para transferir información entre diferentes sistemas de bases de datos.
- LOAD DATA INFILE Es una instrucción de MariaDB que permite importar grandes cantidades de información desde un archivo hacia una tabla.
- INTO OUTFILE Es una instrucción que permite exportar los datos de una tabla a un archivo externo, como CSV.
- COLLATION Es la configuración que determina cómo se comparan y ordenan los textos dentro de una base de datos.
- ETL Significa extraer, transformar y cargar. Es un proceso que permite obtener información de una base de datos, modificarla y trasladarla a otro sistema.
- ODBC Es una tecnología que permite conectar diferentes aplicaciones con sistemas de bases de datos.
- AWS DMS Es un servicio de Amazon que permite migrar bases de datos hacia otros servidores o servicios en la nube.
- GTID Es un identificador que permite reconocer las transacciones realizadas y facilita la replicación entre servidores.

## link github

https://github.com/jorgecrd97-eng/PRACTICAS