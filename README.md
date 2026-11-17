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



