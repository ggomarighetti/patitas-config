# Configuración de Patitas Felices

Repositorio público de configuración de los sprints 10 a 13 de Microservicios y APIs Escalables.

Config Server obtiene los archivos de la rama `main`. `application.yml` contiene Eureka, Actuator y el marcador `patitas.config-source`. Los archivos `ms-dueno.yml`, `ms-mascota.yml`, `ms-veterinario.yml` y `ms-turno.yml` definen el puerto, la base de datos y la información de cada servicio. `api-gateway.yml` define el punto de entrada en 8080, sus rutas y CORS.

Las credenciales se resuelven con variables de entorno en cada microservicio: `DB_USERNAME` y `DB_PASSWORD`. `DB_URL` y `EUREKA_URL` permiten sustituir las direcciones locales al ejecutar con Docker. No se almacena una contraseña real en Git.

Config Server consulta este repositorio en `http://localhost:8888/{application}/default` y responde un objeto Environment con las propiedades y el commit de origen. La representación YAML está en `http://localhost:8888/{application}-default.yml`.

Orden de arranque: Config Server (8888), Eureka Server (8761), ms-mascota (8082), ms-veterinario (8083), ms-turno (8084) y api-gateway (8080). ms-dueno (8081) también está disponible. Cada servicio utiliza su propia base de datos y las direcciones de Docker se suministran mediante variables de entorno. Desde el sprint 12 los clientes usan 8080; los puertos de las cuatro APIs permanecen dentro de la red de Docker.

El gateway usa Spring Cloud Gateway Server WebFlux con el prefijo actual spring.cloud.gateway.server.webflux. Las cuatro rutas /api/duenos/**, /api/mascotas/**, /api/veterinarios/** y /api/turnos/** apuntan a destinos lb:// resueltos por Eureka. El alias /v1/duenos/** ejercita RewritePath. El locator genera prefijos en minúsculas para Dueños, Mascotas y Veterinarios. Desde el sprint 13, /ms-turno/** comparte la ruta de Turnos, con RewritePath y el mismo balde de tokens, para que cambiar al alias no eluda su cuota. Se excluyen del locator Turnos y el propio gateway: hay ocho rutas en total.

CORS de desarrollo acepta cualquier origen sin credenciales, incluyendo PATCH para conservar la cancelación de turnos. Actuator del gateway expone health, info y gateway, con acceso de rutas read-only. RewriteLocationResponseHeader conserva el host y puerto públicos en las cabeceras Location de las altas.

Desde el sprint 11, ms-turno obtiene los timeouts de OpenFeign desde ms-turno.yml: conexión de 3000 ms y lectura de 5000 ms, compresión de solicitud/respuesta habilitada y logger BASIC con DEBUG para el paquete de clientes. Se usa el prefijo vigente spring.cloud.openfeign; el prefijo feign.client del ejemplo de la consigna ya no configura estos clientes. Los destinos se resuelven por nombre mediante Eureka; no se fija una URL para cada cliente.

Referencia: [propiedades de Spring Cloud OpenFeign](https://docs.spring.io/spring-cloud-openfeign/reference/spring-cloud-openfeign.html).

Desde el sprint 13, ms-turno configura los circuitos mascotaService y veterinarioService: ventana COUNT_BASED de 10, mínimo 5 llamadas, umbral 50%, espera OPEN de 10 segundos y tres llamadas de prueba HALF_OPEN. Se habilita la transición automática a HALF_OPEN. El Circuit Breaker envuelve Retry para contar una llamada lógica y rechazar inmediatamente cuando está OPEN. Retry tiene tres intentos totales, esperas de 1 y 2 segundos y un predicado de errores transitorios; los 404 de negocio no se reintentan ni abren circuitos.

Actuator de Turnos expone health, info, metrics, circuitbreakers, circuitbreakerevents y retryevents. El detalle del circuito informa OPEN/CIRCUIT_OPEN o HALF_OPEN/CIRCUIT_HALF_OPEN; el agregador general puede conservar UP. Readiness comprueba el estado de la aplicación y su base, y se usa en Compose para distinguir degradación de dependencia de la disponibilidad del proceso. Los demás servicios mantienen su exposición histórica; los valores de env permanecen ocultos.

El RequestRateLimiter de Gateway usa Redis y limita Turnos por dirección remota de cliente: 10 tokens/segundo, capacidad 20, costo 1. La clave no depende de parámetros ni de cabeceras suministradas por el cliente. Se requiere Redis antes de iniciar Gateway; REDIS_HOST y REDIS_PORT permiten sustituir localhost:6379. Compose usa el nombre redis y mantiene su puerto dentro de la red. Este sprint incorpora el Rate Limiter conforme a su propia consigna; JWT corresponde a una entrega posterior.

Los cambios de configuración se aplican al iniciar el microservicio. Esta entrega no incorpora Spring Cloud Bus ni promete refresco dinámico de propiedades sin reiniciar.

Código e instrucciones: [patitas-felices-Orue-Marighetti](https://github.com/ggomarighetti/patitas-felices-Orue-Marighetti).
