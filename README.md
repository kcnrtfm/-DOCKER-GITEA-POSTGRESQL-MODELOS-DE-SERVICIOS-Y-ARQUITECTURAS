# Arquitectura de servicios: Docker, Gitea y PostgreSQL

## Introducción

Cuando desplegamos una aplicación, no solamente debemos considerar cómo instalarla y ponerla en funcionamiento. También necesitamos comprender qué componentes intervienen, cómo se relacionan entre sí, qué recursos necesitan y qué responsabilidades cumple cada uno dentro del sistema.

Estas cuestiones forman parte del diseño de una arquitectura de servicios.

En este laboratorio vamos a estudiar algunos conceptos fundamentales de arquitectura utilizando un caso concreto: **el despliegue de Gitea, una plataforma de gestión de repositorios Git, con PostgreSQL como sistema de gestión de bases de datos.**

El propósito no es simplemente conseguir que ambas aplicaciones funcionen, sino comprender las decisiones arquitectónicas que intervienen en su despliegue.

A partir de este escenario analizaremos cómo separar responsabilidades entre servicios, establecer mecanismos de comunicación, administrar la persistencia de los datos y organizar los recursos necesarios para ejecutar una aplicación.

Utilizaremos Docker y Docker Compose para implementar la arquitectura, pero es importante comprender que **los contenedores son una tecnología de implementación y no constituyen, por sí mismos, una arquitectura de software**.

Los principios que estudiaremos pueden aplicarse también a servicios instalados directamente sobre sistemas GNU/Linux, máquinas virtuales y otras plataformas de infraestructura.

A lo largo del laboratorio construiremos una arquitectura funcional y observaremos su comportamiento para relacionar los conceptos teóricos con una implementación real.

**El objetivo es aprender a pensar y analizar una arquitectura de servicios, no solamente a ejecutar los comandos necesarios para desplegarla.**

## Fundamentos de arquitectura de sistemas

### **¿Qué es la arquitectura de sistemas?**

La **arquitectura de sistemas** es la disciplina que estudia y define cómo se organiza un sistema informático, identificando sus componentes, las responsabilidades de cada uno y las relaciones que establecen entre sí.

Su propósito es diseñar soluciones que respondan a determinados requisitos funcionales y no funcionales, considerando aspectos como el rendimiento, la disponibilidad, la seguridad, la escalabilidad y el mantenimiento.

La arquitectura no se limita a seleccionar tecnologías o herramientas. También comprende las decisiones sobre cómo distribuir las funciones del sistema, cómo establecer la comunicación entre sus componentes y cómo organizar los recursos de infraestructura necesarios para su funcionamiento.

Diseñar una arquitectura implica responder preguntas como:

- ¿Qué componentes necesita el sistema?
- ¿Qué función cumple cada componente?
- ¿Cómo se comunican entre sí?
- ¿Dónde se ejecutan los servicios?
- ¿Cómo se almacenan y protegen los datos?
- ¿Qué sucede cuando un componente deja de funcionar?
- ¿Cómo puede mantenerse, modificarse o ampliarse la infraestructura?

Estas preguntas permiten comprender que una arquitectura no consiste solamente en seleccionar tecnologías, sino también en definir cómo se organiza y funciona el conjunto.

Además la arquitectura de sistemas tiene un impacto directo sobre la **seguridad**, ya que determina cómo se distribuyen los servicios, qué recursos pueden utilizar y cómo se comunican entre sí. Decisiones como la separación de componentes, el aislamiento de procesos, la segmentación de redes y la aplicación del principio de mínimos privilegios permiten reducir la superficie de ataque y limitar las consecuencias de una posible vulnerabilidad. Por lo tanto, la seguridad no depende únicamente de la configuración individual de cada servicio, sino también de las decisiones arquitectónicas que definen el funcionamiento del sistema en su conjunto.

**En nuestro laboratorio**, utilizaremos Gitea y PostgreSQL para estudiar estas decisiones y comprender cómo un diseño arquitectónico se transforma en una implementación concreta.



## Laboratorio: Gitea y PostgreSQL

### Presentación del laboratorio

En este laboratorio vamos a utilizar Gitea y PostgreSQL como caso de estudio para analizar el diseño de una arquitectura de servicios.

Antes de definir cómo y dónde ejecutar las aplicaciones, necesitamos conocer sus características, requisitos técnicos y dependencias. Esto nos permitirá identificar los componentes necesarios, las relaciones entre ellos y las condiciones que debe satisfacer la infraestructura.

### Gitea

Gitea es una plataforma de código abierto para alojar y administrar repositorios Git. Proporciona una interfaz web para gestionar repositorios, usuarios, organizaciones, permisos y proyectos, además de permitir operaciones Git mediante HTTP/HTTPS y SSH.

Desde el punto de vista arquitectónico, Gitea representa el servicio de aplicación.

**Requisitos y dependencias:**

- **Procesamiento y memoria:** recursos suficientes para ejecutar la aplicación y atender las operaciones de los usuarios.
- **Git:** necesario para las operaciones sobre repositorios.
- **Base de datos:** almacenamiento de información estructurada, como usuarios, permisos y metadatos de los repositorios. Gitea admite SQLite, PostgreSQL, MySQL y MariaDB.
- **Sistema de archivos:** almacenamiento persistente de repositorios Git, archivos de configuración y otros datos de la aplicación.
- **Conectividad:** acceso mediante HTTP/HTTPS para la interfaz web y las operaciones Git, y opcionalmente SSH.
- **Permisos:** una identidad de ejecución con acceso a los recursos necesarios.

Referencia: https://docs.gitea.com/

### PostgreSQL

PostgreSQL es un sistema de gestión de bases de datos relacionales de código abierto. Permite almacenar, consultar y modificar información mediante SQL, proporcionando mecanismos de integridad, concurrencia y administración de transacciones.

En nuestra arquitectura, PostgreSQL será el servicio responsable de administrar los datos estructurados utilizados por Gitea.

**Requisitos y dependencias:**

- **Procesamiento y memoria:** recursos necesarios para ejecutar el motor de base de datos y atender consultas concurrentes.
- **Almacenamiento persistente:** espacio para los archivos de datos y registros de transacciones.
- **Conectividad:** mecanismos de comunicación que permitan recibir conexiones de la aplicación, habitualmente mediante TCP/IP.
- **Autenticación:** identificación de los clientes que intentan acceder al servicio.
- **Autorización:** administración de usuarios, roles y permisos sobre las bases de datos.
- **Integridad y recuperación:** mecanismos para proteger la consistencia de la información y permitir su recuperación ante fallos.

Referencia: https://www.postgresql.org/docs/

### Dependencias entre los servicios

Para que Gitea utilice PostgreSQL, necesitamos resolver una serie de condiciones:

- **Comunicación:** Gitea debe poder establecer una conexión con PostgreSQL.
- **Identificación:** la aplicación necesita conocer la dirección y el puerto del servicio de base de datos.
- **Autenticación y permisos:** PostgreSQL debe disponer de una base de datos y una identidad autorizada para las operaciones de Gitea.
- **Disponibilidad:** debemos contemplar el comportamiento de la aplicación cuando la base de datos no está disponible.
- **Persistencia:** tanto los repositorios Git como los datos administrados por PostgreSQL deben conservarse independientemente de los procesos que ejecutan los servicios.

Estas dependencias existen independientemente de la tecnología utilizada para desplegar la solución. Resolverlas forma parte del diseño arquitectónico que desarrollaremos durante el laboratorio.

### Planteamiento del diseño arquitectónico

Una vez identificados los componentes, requisitos y dependencias de Gitea y PostgreSQL, debemos determinar cómo organizar e implementar nuestra solución.

Para ello, necesitamos tomar una serie de decisiones arquitectónicas. No existe una única manera de implementar un sistema: podemos utilizar diferentes tecnologías, modelos de servicios e infraestructuras para satisfacer los mismos requisitos.

Antes de seleccionar las herramientas, vamos a plantearnos algunas preguntas.

#### 1. ¿Qué servicios necesitamos y cómo vamos a proporcionarlos?

Nuestra solución requiere una aplicación, una base de datos, almacenamiento y mecanismos de comunicación.

Sin embargo, no todos estos componentes deben necesariamente instalarse y administrarse por nuestra cuenta.

Por ejemplo, PostgreSQL podría implementarse como un servicio instalado directamente sobre un servidor, ejecutarse dentro de un contenedor o consumirse como un servicio de base de datos administrado por un proveedor (DBaaS).

Cada alternativa implica diferentes responsabilidades, niveles de control y costes.

#### 2. ¿Cómo organizamos los componentes y sus relaciones?

Necesitamos establecer qué responsabilidad tendrá cada componente y cómo se relacionará con los demás.

Una arquitectura puede incluir servidores web, aplicaciones, bases de datos, almacenamiento y servicios externos.

Debemos identificar las dependencias entre estos elementos, los mecanismos de comunicación y los recursos que necesitan compartir.

También necesitamos una forma de representar estas relaciones, mediante diagramas y documentación que permitan comprender el funcionamiento del conjunto.

#### 3. ¿Dónde vamos a ejecutar los servicios?

Podemos instalar los componentes sobre un mismo sistema operativo, distribuirlos entre distintos servidores, utilizar máquinas virtuales o ejecutarlos mediante contenedores.

También podemos combinar infraestructura propia con servicios proporcionados por terceros.

Estas alternativas presentan diferencias en aislamiento, rendimiento, disponibilidad, administración y costes.

#### 4. ¿Cómo administramos las dependencias y configuraciones?

Cada aplicación necesita determinados componentes para funcionar: bibliotecas, archivos de configuración, herramientas y versiones específicas de software.

Debemos decidir cómo instalar y mantener estas dependencias, evitar conflictos entre aplicaciones y garantizar que el entorno pueda reproducirse.

Este problema adquiere mayor importancia cuando necesitamos desplegar varios servicios o mantener diferentes versiones de una misma aplicación.

#### 5. ¿Cómo aislamos y protegemos los servicios?

Debemos determinar qué recursos puede utilizar cada componente y qué mecanismos permiten limitar su acceso al resto del sistema.

Podemos emplear usuarios y permisos del sistema operativo, aislamiento de procesos, máquinas virtuales, contenedores y mecanismos de segmentación de redes.

Estas decisiones influyen directamente sobre la seguridad y sobre las consecuencias que podría tener el compromiso de uno de los componentes.

#### 6. ¿Cómo administramos el ciclo de vida de los servicios?

Un servicio necesita mecanismos que permitan iniciarlo, detenerlo, reiniciarlo, actualizarlo y recuperar su funcionamiento ante fallos.

Tradicionalmente, en GNU/Linux utilizamos administradores de servicios como `systemd` para controlar procesos que proporcionan funcionalidades, por ejemplo, un servidor web o un motor de base de datos.

En otros modelos de implementación, como las plataformas de contenedores, estas responsabilidades se administran mediante mecanismos diferentes.

Debemos comprender qué significa ejecutar un servicio y cómo cambia su administración según el entorno elegido.

#### 7. ¿Cómo resolvemos la comunicación y la persistencia?

Nuestros componentes necesitan intercambiar información y conservar determinados datos.

En el caso de Gitea y PostgreSQL, debemos garantizar que la aplicación pueda conectarse con la base de datos y que la información permanezca disponible aunque los procesos se detengan o se reemplacen.

Esto implica tomar decisiones sobre redes, almacenamiento, permisos de acceso, copias de seguridad y recuperación.

#### 8. ¿Cómo describimos y reproducimos nuestra arquitectura?

Finalmente, necesitamos establecer cómo documentar e implementar las decisiones tomadas.

Podemos configurar cada componente manualmente o utilizar herramientas que permitan describir de manera declarativa los servicios, sus dependencias y los recursos necesarios para ejecutarlos.

Este enfoque permite automatizar despliegues, mantener configuraciones reproducibles y facilitar la administración de sistemas compuestos por varios servicios.

### El coste como criterio de diseño

Todas estas decisiones tienen consecuencias técnicas y económicas.

No debemos considerar únicamente el coste de adquirir o contratar infraestructura, sino también el esfuerzo necesario para instalarla, administrarla, mantenerla y protegerla.

Un servicio administrado puede tener un coste directo mayor, pero reducir las tareas operativas. Una infraestructura autogestionada puede ofrecer mayor control, aunque requiera más conocimientos, tiempo y recursos de administración.

Por lo tanto, **el diseño arquitectónico consiste también en evaluar compromisos entre funcionalidad, seguridad, disponibilidad, complejidad y coste**.

A partir de estas preguntas, podremos analizar diferentes alternativas y seleccionar un modelo de implementación para nuestro laboratorio, justificando las decisiones que nos conduzcan a una arquitectura basada en contenedores.

### Arquitectura tradicional con servidores dedicados

A partir de los requisitos identificados para Gitea y PostgreSQL, podemos considerar una arquitectura cliente-servidor tradicional, basada en la separación de la aplicación y su base de datos.

Este modelo representa una solución habitual en infraestructuras de sistemas, donde los servicios se distribuyen entre servidores independientes para separar sus responsabilidades, recursos y entornos de ejecución.

Nuestra arquitectura está compuesta por dos servidores GNU/Linux conectados a una misma red local `192.168.10.0/24`.

El **servidor de aplicaciones**, con dirección `192.168.10.10`, ejecuta Gitea junto con Git y sus dependencias. Proporciona acceso a la plataforma mediante HTTP/HTTPS y permite realizar operaciones sobre los repositorios mediante SSH. Los repositorios, archivos de configuración y demás datos de la aplicación se encuentran almacenados en su sistema de archivos.

El **servidor de base de datos**, con dirección `192.168.10.20`, ejecuta PostgreSQL. Su responsabilidad es administrar la información estructurada utilizada por Gitea. Dispone de almacenamiento persistente propio para conservar los archivos de la base de datos y sus registros de transacciones.

La comunicación entre ambos servidores se realiza mediante TCP/IP. Gitea establece conexiones con PostgreSQL utilizando el puerto 5432, mientras que los usuarios interactúan únicamente con el servidor de aplicaciones. La base de datos no se encuentra expuesta directamente a los clientes.

Esta separación permite administrar los servicios de manera independiente y aplicar restricciones de acceso específicas para cada componente.

En ambos servidores, las aplicaciones se ejecutan como servicios del sistema operativo, administrados mediante `systemd`. Cada entorno mantiene sus propias dependencias, configuraciones, usuarios y permisos.

La arquitectura incorpora además un destino de copias de seguridad independiente, donde se conservan respaldos de los repositorios Git, la configuración de Gitea y los datos de PostgreSQL, permitiendo recuperar la información ante posibles fallos.

Desde el punto de vista operativo, este modelo implica mantener dos sistemas operativos, administrar sus recursos de procesamiento y memoria, garantizar la conectividad de red y gestionar el almacenamiento y las copias de seguridad.

Estas responsabilidades representan también costes de infraestructura y administración, que forman parte de las decisiones arquitectónicas.

Este modelo puede implementarse tanto sobre servidores físicos como en entornos virtualizados. Se trata de una solución válida y consolidada, que permite separar responsabilidades y administrar los componentes de manera independiente.

**La organización de esta infraestructura responde a un patrón arquitectónico ampliamente utilizado**, basado en la separación de responsabilidades, la especialización de los componentes y la comunicación mediante interfaces definidas.

No es casualidad que encontremos estructuras similares en numerosos sistemas informáticos. Aunque cambien las aplicaciones, las tecnologías o la cantidad de servidores, las necesidades de procesamiento, almacenamiento y comunicación siguen presentes.

Por eso, las arquitecturas tienden a reproducir modelos conocidos que han demostrado ser eficaces para resolver problemas recurrentes. **Lo que permanece es el patrón arquitectónico; lo que puede variar es la forma de implementarlo.**

### Despliegue manual de Gitea y PostgreSQL con Docker

Vamos a reproducir manualmente la configuración del Compose oficial utilizando comandos Docker. Mantendremos las versiones y los parámetros principales del ejemplo, utilizando volúmenes Docker para la persistencia.

#### 1. Crear la red

```bash
docker network create gitea
```

#### 2. Crear los volúmenes

```bash
docker volume create gitea_data
docker volume create postgres_data
```

#### 3. Descargar las imágenes

```bash
docker pull docker.gitea.com/gitea:28.4.0
docker pull postgres:18
```

#### 4. Crear el contenedor PostgreSQL

```bash
docker run -d \
  --name db \
  --restart always \
  --network gitea \
  -e POSTGRES_USER=gitea \
  -e POSTGRES_PASSWORD=gitea \
  -e POSTGRES_DB=gitea \
  -v postgres_data:/var/lib/postgresql \
  postgres:18
```

#### 5. Verificar PostgreSQL

```bash
docker ps
```

```bash
docker exec db pg_isready -U gitea -d gitea
```

#### 6. Crear el contenedor Gitea

```bash
docker run -d \
  --name gitea \
  --restart always \
  --network gitea \
  -p 3000:3000 \
  -p 222:22 \
  -e USER_UID=1000 \
  -e USER_GID=1000 \
  -e GITEA__database__DB_TYPE=postgres \
  -e GITEA__database__HOST=db:5432 \
  -e GITEA__database__NAME=gitea \
  -e GITEA__database__USER=gitea \
  -e GITEA__database__PASSWD=gitea \
  -v gitea_data:/data \
  -v /etc/timezone:/etc/timezone:ro \
  -v /etc/localtime:/etc/localtime:ro \
  docker.gitea.com/gitea:28.4.0
```

#### 7. Verificar los contenedores

```bash
docker ps
```

```bash
docker logs gitea
docker logs db
```

#### 8. Inspeccionar la red

```bash
docker network inspect gitea
```

#### 9. Inspeccionar los volúmenes y sus datos

```bash
docker volume inspect gitea_data
docker volume inspect postgres_data
```

Para conocer los directorios reales donde Docker almacena los datos en el anfitrión, consultamos el campo `Mountpoint`.

```bash
docker volume inspect -f '{{ .Mountpoint }}' gitea_data
docker volume inspect -f '{{ .Mountpoint }}' postgres_data
```

En una instalación habitual de Docker Engine con permisos de administrador sobre GNU/Linux, estos directorios suelen ser:

```text
/var/lib/docker/volumes/gitea_data/_data
/var/lib/docker/volumes/postgres_data/_data
```

Podemos listar su contenido usando las rutas devueltas por `docker volume inspect`, sin asumir que Docker utiliza siempre `/var/lib/docker`.

```bash
sudo ls -la "$(docker volume inspect -f '{{ .Mountpoint }}' gitea_data)"
sudo ls -la "$(docker volume inspect -f '{{ .Mountpoint }}' postgres_data)"
```

En el interior de los contenedores, los volúmenes están montados en `/data` para Gitea y `/var/lib/postgresql` para PostgreSQL. La ubicación real en el anfitrión puede variar según la configuración de Docker y el modo de ejecución.

#### 10. Acceder a Gitea

Desde un navegador web:

```text
http://IP_DEL_SERVIDOR:3000
```

Completar la instalación inicial de Gitea.

**Resultado.** Dos contenedores independientes conectados mediante una red virtual, con almacenamiento persistente y comunicación interna entre Gitea y PostgreSQL.

La contraseña `gitea` reproduce el ejemplo de laboratorio y no debe utilizarse en producción. No se publica el puerto de PostgreSQL en el anfitrión.
