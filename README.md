# Stepia Backend

Este repositorio contiene el diseño de arquitectura y el desarrollo del backend para una aplicación de celular de geolocalización (en estilo videojuego), inspirada en conceptos como *Fog of World*. El objetivo de la app es permitir a los usuarios explorar el mundo real, "desbloquear" áreas geográficas, realizar un seguimiento de su progreso por ciudades (buscando alcanzar el 100% de exploración) y competir en tiempo real con amigos a través de tablas de clasificación (*leaderboards*).

> **Estado del Proyecto:** En fase de diseño de arquitectura y modelado de datos.

---

## Stack Tecnológico Planeado

La selección de tecnologías está orientada a construir un sistema robusto, escalable y preparado para el manejo eficiente de datos espaciales:

- **Lenguaje:** Java 17+
- **Framework Principal:** Spring Boot (Spring Web, Spring Security)
- **Base de Datos:** PostgreSQL (Ideal para la persistencia relacional de usuarios y relaciones de amistad).
- **Indexación Geoespacial:** **Uber H3 (H3-Java)**. En lugar de procesar polígonos tradicionales pesados, se utiliza el sistema de celdas hexagonales de Uber para discretizar el mapa mundial. Esto permite que el cálculo de áreas exploradas se reduzca a operaciones de conjuntos sumamente eficientes.
- **Persistencia/ORM:** Spring Data JPA / Hibernate.
- **Autenticación:** JSON Web Tokens (JWT) para una arquitectura REST stateless.
- **Gestor de Dependencias:** Gradle.

---

## Diseño de Arquitectura y Capas

El backend se estructurará siguiendo los principios de la **Arquitectura Multicapa (Controller-Service-Repository)** para garantizar un bajo acoplamiento y facilitar el testing unitario:

1. **API/Controller Layer:** Exposición de endpoints RESTful en formato JSON.
2. **Service Layer:** Contendrá la lógica de negocio principal (cálculo de porcentajes de ciudades exploradas, validación de rutas cruzando datos con los límites de la ciudad, y procesamiento de rankings).
3. **Repository/Data Access Layer:** Consultas optimizadas a la base de datos relacional aprovechando los índices espaciales de PostGIS.

---

## Modelo de Datos (Entidades Principales)

Para soportar las funcionalidades de gamificación y geolocalización, se definieron las siguientes entidades core:

- **User:** ID, username, email, password (hashed), total_score, created_at.
- **City:** ID, name, bounding_box_coordinates.
- **ExploredCell (H3 Indexes):** user_id, h3_index (String/Long representando el hexágono visitado, típicamente en resolución 8 o 9 para precisión a nivel de calles), visited_at.
- **UserProgress:** ID, user_id, city_id, percentage_explored (Calculado de manera eficiente dividiendo la cantidad de `h3_index` únicos del usuario dentro de la ciudad sobre el total de hexágonos que componen la ciudad completa).
- **Friendship:** ID, user_id_1, user_id_2, status (PENDING, ACCEPTED).

---

## Diseño de la API REST (Endpoints Core)

A continuación se detallan los principales endpoints diseñados para la comunicación con la aplicación cliente:

### Autenticación y Usuarios
- `POST /api/v1/auth/register` - Registro de nuevos usuarios.
- `POST /api/v1/auth/login` - Autenticación y devolución de token JWT.
- `GET /api/v1/users/profile` - Obtener información del perfil del usuario autenticado.

### Geolocalización y Progreso
- `POST /api/v1/routes` - Sincronizar una nueva ruta de coordenadas GPS recolectada por el dispositivo móvil.
- `GET /api/v1/progress/cities` - Listar las ciudades visitadas por el usuario y su porcentaje de exploración.
- `GET /api/v1/progress/cities/{cityId}` - Detalle geométrico de las zonas específicas exploradas dentro de una ciudad.

### Social y Gamificación (Competencia)
- `POST /api/v1/friends/request/{targetUserId}` - Enviar solicitud de amistad.
- `GET /api/v1/friends/leaderboard` - Obtener la tabla de posiciones en tiempo real comparando el puntaje del usuario con el de sus amigos.

---

## Próximos Pasos (Roadmap de Desarrollo)

1. [ ] Inicialización del proyecto base con Spring Initializr y configuración del contenedor Docker para PostgreSQL + PostGIS.
2. [ ] Implementación del módulo de usuarios, seguridad con JWT y migraciones de base de datos básicas.
3. [ ] Integración de la librería `h3-java` para convertir las coordenadas GPS (`latitude, longitude`) recibidas en tiempo real a sus respectivos índices hexagonales H3.
4. [ ] Desarrollo de la lógica de negocio para la agregación de hexágonos descubiertos y actualización óptima de los leaderboards compartidos.
