# Informe sobre la Instalación de SQL Server con Docker
## Objetivo
El objetivo de este informe es detallar el proceso de instalación y configuración de SQL Server utilizando Docker en un sistema Windows, desde la instalación del contenedor hasta la creación de una base de datos y la visualización de la misma a través de Azure Data Studio.
## Requisitos previos
- Tener Docker instalado en tu sistema.
- Tener acceso a PowerShell o una terminal de tu preferencia.
- Si no tienes WSL (Windows Subsystem for Linux) habilitado, se explica cómo hacerlo en los siguientes pasos.
## Pasos para la instalación
### 1. Verificación de WSL (si no está instalado)
Si necesitas usar WSL para ejecutar herramientas de Linux como sqlcmd dentro del contenedor de Docker, pero no tienes WSL habilitado, puedes instalarlo siguiendo estos pasos:

#### 1.1. Habilitar WSL en Windows

Abre PowerShell como Administrador.

Ejecuta el siguiente comando para habilitar WSL:
Abre PowerShell como Administrador y ejecuta el siguiente comando para habilitar WSL:

powershell
wsl --install
Esto instalará WSL y descargará la distribución predeterminada (Ubuntu).


Esto instalará WSL y descargará la distribución predeterminada (Ubuntu). Después de ejecutar este comando, reinicia tu PC si se te solicita.

Después de ejecutar este comando, reinicia tu PC si se te solicita.
#### 1.2. Verificación de WSL

1.2. Verificación de WSL
Una vez reiniciado, puedes verificar que WSL está instalado correctamente ejecutando el siguiente comando en PowerShell:

powershell
powershell
wsl -l -v


Este comando te mostrará las distribuciones instaladas de WSL en tu sistema. Si aún no tienes una distribución instalada, puedes obtenerla desde la Microsoft Store, por ejemplo, Ubuntu o Debian.

1.3. Actualización de WSL (si es necesario)
#### 1.3. Actualización de WSL (si es necesario)

Si ya tienes WSL instalado y necesitas actualizarlo a WSL 2 (recomendado para un mejor rendimiento), ejecuta el siguiente comando en PowerShell como administrador:

powershell
powershell
wsl --set-default-version 2


Esto configurará WSL 2 como la versión predeterminada para nuevas distribuciones.

2. Instalación de Docker
### 2. Instalación de Docker

Si aún no tienes Docker instalado, descárgalo e instálalo desde su sitio oficial. Una vez instalado Docker, asegúrate de que esté funcionando correctamente ejecutando el siguiente comando en PowerShell:

powershell
powershell
docker --version
3. Descarga de la Imagen de SQL Server


### 3. Descarga de la Imagen de SQL Server

Para usar SQL Server en Docker, necesitas descargar la imagen oficial de Microsoft SQL Server desde Docker Hub. El siguiente comando descarga la imagen más reciente de SQL Server 2022:

powershell
powershell
docker pull mcr.microsoft.com/mssql/server:2022-latest



### 4. Crear y Ejecutar el Contenedor de SQL Server

Para crear un contenedor de SQL Server, ejecuta el siguiente comando en PowerShell. Esto creará un contenedor con el nombre sqlserver y expondrá el puerto 1433 (puerto predeterminado de SQL Server):

powershell
docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=yourStrong#Password" -p 1433:1433 --name sqlserver -d mcr.microsoft.com/mssql/server:2022-latest


*Explicación de los parámetros:*
- -e "ACCEPT_EULA=Y": Acepta el contrato de licencia de SQL Server.
- -e "SA_PASSWORD=yourStrong#Password": Establece la contraseña para el usuario sa (administrador del sistema).
- -p 1433:1433: Mapea el puerto 1433 del contenedor al puerto 1433 de tu máquina local.
- --name sqlserver: Le asigna el nombre sqlserver al contenedor.
- -d: Ejecuta el contenedor en segundo plano (modo "detached").

### 5. Verificación de la Ejecución del Contenedor

Para verificar que el contenedor está en ejecución, usa el siguiente comando:

powershell
powershell
docker ps


Este comando te mostrará los contenedores en ejecución y te confirmará que SQL Server está corriendo correctamente.

6. Acceso a SQL Server desde el Contenedor
### 6. Acceso a SQL Server desde el Contenedor

Para conectarte a SQL Server y ejecutar comandos SQL, utiliza el siguiente comando para entrar al contenedor:

powershell
powershell
docker exec -it sqlserver bash
Dentro del contenedor, ejecuta el siguiente comando para acceder a SQL Server utilizando la herramienta sqlcmd:


Dentro del contenedor, verifica si sqlcmd está instalado ejecutando:

bash
bash
command -v sqlcmd


Si este comando no devuelve nada, significa que sqlcmd no está instalado. Para instalarlo, sigue estos pasos:

1. Sal del contenedor con exit.
2. Vuelve a entrar al contenedor, pero esta vez como usuario root:

   powershell
   docker exec -it --user root sqlserver bash
   

3. Instala sqlcmd dentro del contenedor ejecutando:

   bash
   apt-get update
   apt-get install -y mssql-tools unixodbc-dev
   

4. Sal del usuario root con exit.
5. Vuelve a entrar al contenedor como usuario normal:

   powershell
   docker exec -it sqlserver bash
   

Ahora puedes ejecutar sqlcmd sin problemas:

bash
/opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P yourStrong#Password
Explicación:
-S localhost: Se conecta al servidor SQL que está corriendo en el mismo contenedor.
-U sa: El usuario administrador de SQL Server.
-P yourStrong#Password: La contraseña que definiste para el usuario sa.


### 7. Creación de la Base de Datos

7. Creación de la Base de Datos
Una vez dentro de SQL Server, puedes crear una base de datos ejecutando el siguiente comando SQL:
sql

sql
CREATE DATABASE GameLibrary;
Esto creará una base de datos llamada GameLibrary.


Esto creará una base de datos llamada GameLibrary.

### 8. Creación de una Tabla en la Base de Datos

8. Creación de una Tabla en la Base de Datos
Para trabajar con datos en la base de datos, primero cambia el contexto a la base de datos recién creada:
sql

sql
USE GameLibrary;


A continuación, crea una tabla Games:
sql
A continuación, crea una tabla Games:

sql
CREATE TABLE Games (
    GameID INT PRIMARY KEY IDENTITY(1,1),
    Title NVARCHAR(100) NOT NULL,
    Genre NVARCHAR(50) NOT NULL,
    ReleaseDate DATE
);
9. Inserción de Datos en la Tabla
Para insertar datos en la tabla Games, usa el siguiente comando SQL:
sql


### 9. Inserción de Datos en la Tabla

Para insertar datos en la tabla Games, usa el siguiente comando SQL:

sql
INSERT INTO Games (Title, Genre, ReleaseDate)
VALUES ('Valorant', 'Shooter', '2020-06-02'),
       ('Fortnite', 'Battle Royale', '2017-07-25');
10. Verificación de los Datos Insertados


### 10. Verificación de los Datos Insertados

Puedes consultar los datos insertados con el siguiente comando SQL:
sql

sql
SELECT * FROM Games;
Esto te mostrará los datos de los juegos que acabas de agregar.



### 11. Uso de Azure Data Studio para Visualizar la Base de Datos

Para facilitar la administración y visualización de la base de datos, puedes usar Azure Data Studio.

1. Descarga e instala Azure Data Studio.
2. Abre Azure Data Studio.
3. En la ventana de Conexión, ingresa los siguientes detalles:
   - *Servidor:* localhost,1433
   - *Autenticación:* SQL Login
   - *Usuario:* sa
   - *Contraseña:* yourStrong#Password
4. Haz clic en *Conectar*.

### Conclusión

En este informe se detalla todo el proceso para instalar y configurar SQL Server en Docker en un sistema Windows, asegurando que sqlcmd esté instalado correctamente antes de proceder con la gestión de la base de datos.
