# Registro de trabajo con IA

## Prompt inicial

> Actúa como desarrollador Full-Stack Senior en Java y especialista en Docker. Construye BikeStore para la práctica de Acceso a Datos (2.º CFGS DAM), con Java 17, Maven WAR, Tomcat 10.1, MySQL 8.0, JDBC/HikariCP, Jackson, JUnit 5, Mockito, H2 y Adminer. Mantén separación estricta por capas y configuración por variables de entorno; prepara la interfaz CRUD, el flujo transaccional de pedidos, Docker Compose, pruebas y documentación.

## Decisiones arquitectónicas

- API REST JSON con Jakarta Servlets y Jackson: Tomcat 10.1 incluye Servlet, pero no ofrece un runtime JAX-RS por defecto.
- Interfaz HTML/CSS/JavaScript vanilla; la presentación consume la API mediante `fetch`.
- Modelos internos separados de DTOs, JDBC mediante `PreparedStatement` y recursos cerrados con `try-with-resources`.
- Los servicios calculan los totales y gestionan stock en transacciones JDBC; H2 en modo MySQL mantiene las pruebas aisladas.
- Las bicicletas con histórico se desactivan para conservar integridad referencial.
- Flyway versiona el esquema y los cambios de datos; HikariCP proporciona el pool de conexiones.
- Los errores HTTP se traducen a respuestas JSON con códigos semánticos; las reglas de negocio y la validación se mantienen en el servidor.

## Cómo se guio al agente

- Se trabajó de forma iterativa: primero se pidió adaptar la página a móviles; después se solicitó revisar los criterios de persistencia y arquitectura, proponer faltantes e implementar las mejoras aceptadas.
- A continuación se pidió contrastar capas, DTOs, configuración externa, validación y códigos HTTP, y aplicar las mejoras recomendadas.
- La última revisión funcional pidió mejorar estados de carga, errores visibles y navegación; la petición actual solicita revisar las pruebas JUnit 5 y documentar el flujo de trabajo asistido.
- El agente revisó los archivos existentes antes de proponer cambios, distinguió requisitos esenciales de mejoras opcionales y ejecutó las pruebas Maven en un contenedor cuando Maven no estaba disponible directamente en el equipo.

## Refactorizaciones y cambios incorporados

Los siguientes cambios constan en el historial de trabajo y en el estado del proyecto. Son cambios realizados con asistencia de Junie a partir de las revisiones y decisiones solicitadas:

- **Persistencia y DAO:** se incorporaron migraciones Flyway versionadas; se añadió unicidad para que una bicicleta no se repita en las líneas de un pedido; `JdbcPedidoDao.findAll()` dejó de cargar líneas con consultas por pedido y las obtiene con una consulta agrupada; `JdbcBicicletaDao.update()` reutiliza la conexión; las líneas guardan el nombre histórico de la bicicleta.
- **Configuración y API:** los parámetros del pool Hikari se pueden configurar por variables de entorno; se validan las URL de imagen; las rutas y métodos no soportados reciben respuestas JSON `404`/`405` y `OPTIONS` responde `204`.
- **Interfaz:** se añadieron estilos responsive, estados accesibles de carga y error con reintento, mensajes de validación en formularios, bloqueo temporal de acciones y navegación con historial.
- **Pruebas:** se amplió `PedidoServiceTest` con cliente/pedido inexistente, líneas repetidas y desbordamiento de cantidad. El resto de pruebas cubre duplicidad de SKU/email/línea de pedido, rollback por falta de stock, cancelación, migraciones y respuestas HTTP.

### Aportaciones manuales del alumno

El historial disponible registra las peticiones y los cambios asistidos, pero no permite identificar qué ediciones o refactorizaciones adicionales hizo manualmente el alumno. Para mantener una atribución honesta, este documento no adjudica al alumno los cambios anteriores. Si realizó cambios propios, debe añadir aquí cada refactorización, el archivo afectado, su motivo y cómo la verificó (idealmente con el commit correspondiente).

## Cobertura y verificación

- La suite usa JUnit Jupiter 5.10.2, Mockito y H2 en modo MySQL; cada prueba crea una base de datos en memoria aislada.
- La cobertura funcional incluye casos exitosos y negativos de DAOs, servicios, migraciones y contrato HTTP. Algunos ejemplos: SKU/email duplicados (`ConflictException`), registros ausentes (`NotFoundException`/`Optional.empty`), pedido con stock insuficiente y rollback, cantidades inválidas o desbordadas, y errores HTTP `400`, `404`, `405` y `500`.
- Última verificación: **32 pruebas aprobadas, 0 fallos, 0 errores y 0 omitidas**; se aplicaron migraciones hasta V3. El proyecto no configura JaCoCo, por lo que este resultado informa escenarios ejecutados, no un porcentaje de cobertura de líneas.

Ejecutar desde PowerShell con Maven instalado:

```powershell
mvn -B test
```

Si Maven no está instalado localmente y Docker está disponible:

```powershell
docker run --rm -v "${PWD}:/workspace" -w /workspace maven:3.9-eclipse-temurin-17 mvn -B test
```

## Incidencias observadas durante la verificación

- La máquina de desarrollo no tenía `mvn` disponible en el `PATH`; ejecutar el mismo comando en la imagen Maven de Docker permitió validar la suite sin instalar herramientas en el host.
- En la ejecución de pruebas se mostraron avisos no bloqueantes de SLF4J (sin implementación de logging, usa NOP) y de class-data sharing de la JVM. No afectaron al resultado: el build terminó correctamente.