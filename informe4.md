# Informe de Práctica: Persistencia y Redes en Docker (Tarea 3)

## 1. Introducción y Objetivos

El objetivo de esta semana fue doble:
1.  **Persistencia:** Demostrar la diferencia entre el almacenamiento temporal del contenedor y el almacenamiento persistente usando Volúmenes de Docker (PostgreSQL).
2.  **Redes:** Crear un entorno de aplicación de múltiples contenedores (MySQL y phpMyAdmin) interconectados mediante una red Docker personalizada.

---

## 2. Práctica 1: Persistencia de Datos (PostgreSQL)

### 2.1. Parte 1: Almacenamiento NO Persistente (Pérdida de Datos)

Se demostró que, sin un volumen, los datos se pierden al eliminar el contenedor.

| **Acción** | **Comando Clave** |
| :--- | :--- |
| **Crear Contenedor** | `docker run --name server_db1 ... postgres:latest` |
| **Datos Creados** | Base de datos `test` y registros en tabla `customer`. |
| **Prueba de Pérdida** | `docker stop server_db1` y `docker rm server_db1`. |
| **Resultado** | La base de datos `test` se perdió al recrear el contenedor. |

### 2.2. Parte 2: Almacenamiento Persistente con Volumen

Se implementó un volumen para asegurar que los datos sobrevivieran al ciclo de vida del contenedor.

| **Acción** | **Comando Clave** |
| :--- | :--- |
| **Crear Volumen** | `docker volume create pgdata` |
| **Crear Contenedor Persistente** | `docker run --name server_db2 ... -v pgdata:/var/lib/postgresql/data postgres:16` |
| **Datos Creados** | Registro de 'Carlos Perez' en la BD `test`. |
| **Prueba de Persistencia** | Se detuvo y eliminó `server_db2`, y se recreó usando el mismo volumen `pgdata`. |
| **Resultado** | La base de datos `test` y sus registros **se conservaron**, confirmando la persistencia. |

---

## 3. Práctica 2: Redes Personalizadas (MySQL y phpMyAdmin)

El objetivo fue crear una red que permitiera la comunicación interna entre los contenedores usando sus nombres de servicio.

### 3.1. Creación de la Red y Contenedores

| **Acción** | **Comando Clave** | **Propósito** |
| :--- | :--- | :--- |
| **1. Red Personalizada** | `docker network create web_db_network` | Aísla los contenedores y permite la comunicación por nombre. |
| **2. Contenedor MySQL** | `docker run --name mysql-server -e MYSQL_ROOT_PASSWORD=midbpass --network web_db_network ...` | Servidor de base de datos conectado a la red. |
| **3. Contenedor phpMyAdmin** | `docker run --name phpmyadmin -p 8080:80 -e PMA_HOST=mysql-server --network web_db_network ...` | Cliente web accesible en `localhost:8080`. **`PMA_HOST=mysql-server`** asegura la conexión interna. |

### 3.2. Conexión y Verificación

1.  **Acceso:** Se accedió a phpMyAdmin vía navegador en `http://localhost:8080`.
2.  **Login:** Se inició sesión exitosamente con `root` y la contraseña `midbpass`, verificando que la red `web_db_network` permitió la conexión entre el contenedor de phpMyAdmin y el contenedor de MySQL a través del nombre del servicio.
3.  **Base de Datos de Prueba:** Se creó la base de datos `test_docker` desde la interfaz web, completando la actividad.

---

## 4. Conclusión General

Ambas prácticas son fundamentales en el desarrollo con Docker:
* La **Persistencia** asegura que los datos críticos sobrevivan, haciendo que los Volúmenes de Docker sean un requisito para bases de datos.
* Las **Redes Personalizadas** permiten construir arquitecturas de microservicios complejas, donde los contenedores se comunican entre sí de manera aislada y robusta usando nombres de servicio, lo cual es más fácil de configurar que las direcciones IP.
