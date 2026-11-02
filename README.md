# Práctica Final de Redes II
## Arquitectura de tres capas en AWS con balanceo de carga

**Curso:** Redes II  
**Proyecto:** Aplicación web de tres capas con balanceo de carga  
**Fecha:** 2 de noviembre de 2026

**Integrantes:**
- Integrante 1: Angela Gabriela Chiyal Pérez
- Integrante 2: Evelyn Jeaneth Guarchaj Tahay
- Integrante 3: Ana Valeria Vicente Axpuac

---

## 1. Descripción del proyecto

El presente proyecto consiste en la implementación de una aplicación web de tres capas utilizando los servicios de Amazon Web Services (AWS). Su propósito es demostrar el funcionamiento de una infraestructura en la nube que incorpora balanceo de carga, escalado automático, seguridad de red y almacenamiento persistente.

La infraestructura fue desplegada mediante una plantilla de AWS CloudFormation, permitiendo crear y administrar los recursos necesarios de forma automatizada.

Durante la práctica se realizaron pruebas de conectividad, distribución de peticiones, rendimiento, escalado automático y tolerancia a fallos.

Finalmente, se personalizó la interfaz web y se implementó una nueva ruta en la API, manteniendo el funcionamiento de los componentes originales.

## 2. Objetivos

### Objetivo general

Implementar y evaluar una arquitectura de tres capas en AWS que permita distribuir peticiones entre diferentes servidores, garantizar la comunicación entre sus componentes y demostrar mecanismos de disponibilidad mediante balanceo de carga y escalado automático.

### Objetivos específicos

- Desplegar la infraestructura mediante AWS CloudFormation.
- Configurar una red VPC con subredes públicas y privadas.
- Implementar un frontend estático utilizando Amazon S3 y CloudFront.
- Ejecutar una API Node.js mediante instancias Amazon EC2.
- Implementar una base de datos PostgreSQL utilizando Amazon RDS.
- Comprobar la distribución de peticiones mediante un Application Load Balancer.
- Evaluar el comportamiento de Auto Scaling ante un aumento de tráfico.
- Simular la falla de una instancia y observar la recuperación del servicio.
- Personalizar la aplicación web y agregar una nueva funcionalidad a la API.

## 3. Arquitectura del sistema

La aplicación está organizada en tres capas principales.

### 3.1. Capa de presentación

La capa de presentación está desarrollada utilizando HTML, CSS y JavaScript.

Los archivos se almacenan en un bucket privado de Amazon S3 y se distribuyen mediante Amazon CloudFront, proporcionando acceso al sitio web mediante HTTPS.

### 3.2. Capa de aplicación

La capa de aplicación utiliza Node.js y Express, ejecutándose en instancias Amazon EC2.

Las peticiones son distribuidas mediante un Application Load Balancer entre servidores ubicados en diferentes zonas de disponibilidad.

Se utiliza Auto Scaling para mantener al menos dos instancias activas y aumentar la cantidad hasta cuatro cuando las condiciones de carga lo requieren.

### 3.3. Capa de datos

Se utiliza Amazon RDS con PostgreSQL para almacenar información relacionada con las visitas y mensajes registrados por la aplicación.

La base de datos se encuentra en subredes privadas y permite conexiones desde los servidores de la API mediante reglas de seguridad.

### 3.4. Servicios utilizados

| Servicio | Función |
|---|---|
| CloudFormation | Despliegue y administración de la infraestructura |
| Amazon VPC | Administración de la red |
| Amazon S3 | Almacenamiento del frontend |
| CloudFront | Distribución del contenido web |
| Amazon EC2 | Ejecución del backend |
| Application Load Balancer | Distribución de peticiones |
| Auto Scaling | Administración automática de instancias |
| Amazon RDS | Base de datos PostgreSQL |
| CloudWatch | Supervisión y métricas |

## 4. Configuración de red y seguridad

La infraestructura utiliza una VPC con el bloque de direcciones `10.0.0.0/16`.

Se configuraron cuatro subredes distribuidas entre dos zonas de disponibilidad.

| Subred | Dirección CIDR | Tipo |
|---|---|---|
| publica-a | 10.0.1.0/24 | Pública |
| publica-b | 10.0.2.0/24 | Pública |
| privada-a | 10.0.11.0/24 | Privada |
| privada-b | 10.0.12.0/24 | Privada |

Las subredes públicas cuentan con acceso a Internet mediante un Internet Gateway, mientras que las privadas mantienen el acceso restringido a los recursos internos.

### 4.1. Grupos de seguridad

| Grupo | Puerto | Origen permitido |
|---|---|---|
| sg-alb | TCP 80 | Internet |
| sg-api | TCP 3000 | Grupo sg-alb |
| sg-api | TCP 22 | SSHLocation |
| sg-bd | TCP 5432 | Grupo sg-api |

Se realizaron pruebas de conectividad para comprobar que los servicios internos no fueran accesibles directamente desde Internet.

La API pudo comunicarse con PostgreSQL desde las instancias EC2, mientras que la base de datos permaneció protegida mediante las reglas de seguridad.

## 5. Despliegue de la infraestructura

El despliegue se realizó utilizando el archivo `infra-practica-integradora.yaml`.

Se creó una pila llamada `PracticaIntegradora` desde AWS CloudFormation, configurando los parámetros requeridos para la base de datos y las instancias.

Una vez finalizado el proceso, CloudFormation mostró el estado `CREATE_COMPLETE`.

Posteriormente, se subieron los archivos `index.html`, `estilos.css` y `app.js` al bucket S3 creado por la plantilla.

Finalmente, se accedió a la dirección proporcionada por CloudFront y se comprobó que la aplicación podía comunicarse con los servidores EC2 y PostgreSQL.

## 6. Pruebas realizadas

### 6.1. Comprobación de servidores

Se verificó el estado de las instancias pertenecientes al grupo de Auto Scaling.

Inicialmente se encontraron dos instancias en funcionamiento, distribuidas entre diferentes zonas de disponibilidad.

El balanceador reportó los servidores como saludables (`healthy`), confirmando que podían recibir peticiones.

### 6.2. Prueba de balanceo de carga

Se realizaron múltiples solicitudes a la API para observar la distribución de peticiones entre los servidores disponibles.

Las respuestas permitieron identificar diferentes instancias EC2 y zonas de disponibilidad.

Se comprobó que el balanceador distribuía las solicitudes entre los servidores, evitando que todas las peticiones fueran atendidas por una sola instancia.

### 6.3. Prueba de rendimiento con Apache Bench

Se utilizó Apache Bench para realizar una prueba de 2,000 peticiones HTTP con una concurrencia de 20 solicitudes.

El comando utilizado fue:

```bash
ab -n 2000 -c 20 $API
```

**Resultados confirmados:**

| Métrica | Resultado |
|---|---|
| Peticiones solicitadas | 2,000 |
| Concurrencia | 20 |
| Peticiones completadas | 2,000 |
| Peticiones fallidas | 0 |
| Peticiones por segundo | [Completar con resultado de Apache Bench] |
| Tiempo promedio por petición | [Completar] |
| Percentil 95 de respuesta | [Completar] |

Los resultados permitieron comprobar que la aplicación atendió las solicitudes de la prueba sin registrar peticiones fallidas.

### 6.4. Distribución de peticiones en PostgreSQL

Para verificar el reparto de carga, se consultaron los registros almacenados en PostgreSQL.

Se utilizó la siguiente consulta:

```sql
SELECT
    servidor,
    zona,
    COUNT(*) AS peticiones
FROM visitas
GROUP BY servidor, zona
ORDER BY peticiones DESC;
```

En la consulta correspondiente a las últimas 2,000 visitas se obtuvieron los siguientes resultados:

| Servidor | Peticiones | Porcentaje |
|---|---:|---:|
| Servidor A | 1,001 | 50.05 % |
| Servidor B | 999 | 49.95 % |
| **Total** | **2,000** | **100 %** |

Estos resultados evidencian una distribución prácticamente equitativa entre ambas instancias.

### 6.5. Prueba de Auto Scaling

Se generó tráfico artificial hacia la API para provocar un incremento de la carga.

Se utilizó Apache Bench para enviar solicitudes durante un período prolongado, mientras se supervisaban las actividades del grupo de Auto Scaling.

Durante la prueba se observó que la infraestructura aumentó de dos a cuatro instancias saludables.

Esto permitió comprobar el funcionamiento del mecanismo de escalado automático configurado en la plantilla.

Al reducirse la carga, también se supervisó el comportamiento del grupo para comprobar su retorno a la capacidad normal.

### 6.6. Prueba de tolerancia a fallos

Se realizó una prueba de falla deteniendo manualmente una de las instancias EC2 mientras se supervisaba el funcionamiento de la aplicación.

Durante el procedimiento se verificó el estado de los destinos registrados en el balanceador.

La evidencia muestra el cambio de estado de la instancia detenida y la presencia de servidores saludables disponibles para continuar atendiendo solicitudes.

También se supervisó el comportamiento del grupo de Auto Scaling y sus mecanismos de recuperación.

Esta prueba permitió analizar cómo responde la infraestructura cuando uno de sus servidores deja de estar disponible.

## 7. Personalización del proyecto

Como parte de los requisitos de la práctica, se realizaron modificaciones tanto en la interfaz web como en la API.

### 7.1. Personalización del frontend

Se modificaron los archivos:

- `sitio/index.html`
- `sitio/estilos.css`

Se actualizaron los títulos y elementos visuales de la aplicación para adaptarlos a la práctica final de Redes II.

Además, se implementó una nueva combinación de colores utilizando las siguientes paletas:

**Paleta azul:**
- `#EAF2F8`
- `#C9DCE8`
- `#8AA9C4`
- `#3E5C76`
- `#1D2D44`

**Paleta celeste:**
- `#E3F6F5`
- `#A7E0DB`
- `#5FB0C9`
- `#3E6D9C`
- `#2A2F63`

**Paleta rosada:**
- `#EFB5CB`
- `#E586B7`
- `#DD4D92`
- `#9C3A58`
- `#762250`

Se aplicaron gradientes, colores personalizados, tarjetas con bordes redondeados, botones estilizados y mejoras visuales en las secciones de información.

Se conservaron las funcionalidades de consulta de servidores, estadísticas de carga y libro de visitas.

### 7.2. Publicación de los cambios visuales

Los archivos modificados fueron cargados nuevamente en el bucket de Amazon S3.

Posteriormente, se realizó una invalidación en Amazon CloudFront utilizando la ruta:

```text
/*
```

La invalidación finalizó con el estado `Completed`.

Después de este procedimiento se confirmó que el sitio mostraba los nuevos colores y estilos personalizados.

### 7.3. Personalización de la API

Se agregó una ruta HTTP personalizada a la API Node.js:

```text
GET /api/hola
```

Esta ruta devuelve un objeto JSON con información del proyecto y de la instancia que atendió la solicitud.

Código implementado:

```javascript
app.get('/api/hola', (req, res) => {
  res.json({
    mensaje: 'Bienvenido a nuestra práctica final de Redes II',
    proyecto: 'Arquitectura de tres capas en AWS',
    servidor: SERVIDOR,
    zona: ZONA
  });
});
```

La ruta fue incorporada en el código de la API dentro de la plantilla `infra-practica-integradora.yaml`.

Para aplicar los cambios se realizó una actualización de la pila de AWS CloudFormation.

Posteriormente se accedió a `/api/hola` mediante la dirección de CloudFront y se confirmó que la API respondía correctamente.

Esta modificación permitió demostrar que la infraestructura puede actualizarse para incorporar nuevas funcionalidades.

## 8. Rutas de la API

| Método | Ruta | Descripción |
|---|---|---|
| GET | `/api/ping` | Verificación de salud |
| GET | `/api/salud` | Información del servidor y PostgreSQL |
| GET | `/api/visita` | Registro de una visita |
| GET | `/api/estadisticas` | Estadísticas de peticiones |
| GET | `/api/mensajes` | Consulta de mensajes |
| POST | `/api/mensajes` | Registro de mensajes |
| GET | `/api/hola` | Ruta personalizada del grupo |

## 9. Estructura del proyecto

```text
practica-integradora/
├── infra-practica-integradora.yaml
├── sitio/
│   ├── index.html
│   ├── estilos.css
│   └── app.js
├── api/
│   ├── server.js
│   └── package.json
├── comandos-cloudshell.sh
├── consultas-servidor.sh
└── README.md
```

## 10. Estimación de costos

Según la guía de la práctica, se estimó un costo aproximado de US$0.09 por hora para la infraestructura desplegada.

El costo estimado de mantener los recursos activos durante el período de viernes a lunes era de aproximadamente US$7.

El costo real debe comprobarse mediante AWS Cost Explorer, considerando el tiempo de utilización y los recursos desplegados.

**Costo real registrado:** [Completar con AWS Cost Explorer]

## 11. Evidencias

Se recopilaron capturas correspondientes a:

- Pila CloudFormation creada correctamente.
- Salidas de CloudFormation.
- Sitio web funcionando mediante CloudFront.
- Mapa de recursos de la VPC.
- Subredes y tablas de rutas.
- Grupos de seguridad.
- Pruebas de conectividad.
- Instancias EC2 saludables.
- Consultas en PostgreSQL.
- Distribución de solicitudes entre servidores.
- Resultados de Apache Bench.
- Escalado automático de instancias.
- Prueba de falla de un servidor.
- Interfaz web personalizada.
- Respuesta de `/api/hola`.
- Invalidación completada en CloudFront.

Las evidencias de las primeras fases se encuentran documentadas en el archivo de pruebas elaborado por el equipo.

## 12. Video de demostración

El equipo realizará una presentación en video con una duración aproximada de 6 a 8 minutos.

Durante la demostración se explicará la arquitectura implementada, la configuración de red, las pruebas de balanceo, el escalado automático, la tolerancia a fallos y las modificaciones realizadas.

**Enlace del video:** [Agregar enlace]

## 13. Limpieza de recursos AWS

Una vez finalizadas las pruebas y la demostración, se debe eliminar la infraestructura para evitar cargos adicionales.

El procedimiento consiste en vaciar el bucket de S3 y eliminar la pila `PracticaIntegradora` desde AWS CloudFormation.

Posteriormente debe verificarse que la pila haya alcanzado el estado `DELETE_COMPLETE` y que no permanezcan recursos activos asociados a la práctica.

**Estado de limpieza:** Pendiente de confirmación.

**Evidencia:** [Agregar captura de DELETE_COMPLETE]

## 14. Conclusiones

La práctica permitió implementar una arquitectura de tres capas utilizando servicios de AWS y comprender cómo interactúan los componentes de presentación, aplicación y datos.

Las pruebas realizadas demostraron la distribución de peticiones entre distintas instancias EC2, obteniendo un reparto prácticamente equitativo durante la prueba de 2,000 solicitudes.

Asimismo, se observó el funcionamiento de Auto Scaling ante un incremento de tráfico, alcanzando cuatro instancias saludables.

Las pruebas de conectividad y seguridad permitieron comprender la importancia de separar los recursos mediante subredes y grupos de seguridad.

Finalmente, la personalización del frontend y la incorporación de una nueva ruta en la API demostraron que la aplicación puede modificarse y actualizarse utilizando los servicios implementados.

En conjunto, el proyecto permitió aplicar conocimientos relacionados con redes, infraestructura en la nube, balanceo de carga, disponibilidad y administración de servicios.

---

**Práctica Final — Redes II — Noviembre 2026**
