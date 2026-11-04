# Configuración de Patitas Felices

Repositorio público de configuración de los sprints 10 a 14 de Microservicios y APIs Escalables.

Config Server obtiene los archivos de la rama `main`. `application.yml` contiene Eureka, Actuator y el marcador `patitas.config-source`. Los archivos `ms-dueno.yml`, `ms-mascota.yml`, `ms-veterinario.yml` y `ms-turno.yml` definen el puerto, la base de datos y la información de cada servicio. `api-gateway.yml` define el punto de entrada en 8080, sus rutas y CORS.

Las credenciales se resuelven con variables de entorno en cada microservicio: `DB_USERNAME` y `DB_PASSWORD`. `DB_URL` y `EUREKA_URL` permiten sustituir las direcciones locales al ejecutar con Docker. No se almacena una contraseña real en Git.

Config Server consulta este repositorio en `http://localhost:8888/{application}/default` y responde un objeto Environment con las propiedades y el commit de origen. La representación YAML está en `http://localhost:8888/{application}-default.yml`.

Orden de arranque: Config Server (8888), Eureka Server (8761), ms-auth (8085), las cuatro APIs de negocio y api-gateway (8080). Redis debe estar saludable antes de Gateway. Cada servicio utiliza su propia base y las direcciones de Docker se suministran mediante variables de entorno. Los clientes usan 8080; los puertos 8081–8085 permanecen dentro de la red de Docker.

El gateway usa Spring Cloud Gateway Server WebFlux con el prefijo actual spring.cloud.gateway.server.webflux. Las cuatro rutas de negocio apuntan a destinos lb:// resueltos por Eureka. El alias /v1/duenos/** ejercita RewritePath. El locator genera prefijos para Dueños, Mascotas y Veterinarios. /ms-turno/** comparte la ruta canónica y su cuota. Se excluyen del locator Turnos, Auth y Gateway; con /auth/** hacia lb://ms-auth hay nueve rutas.

CORS de desarrollo acepta cualquier origen sin credenciales, incluyendo PATCH para conservar la cancelación de turnos. Actuator del gateway expone health, info y gateway, con acceso de rutas read-only. RewriteLocationResponseHeader conserva el host y puerto públicos en las cabeceras Location de las altas.

Desde el sprint 11, ms-turno obtiene los timeouts de OpenFeign desde ms-turno.yml: conexión de 3000 ms y lectura de 5000 ms, compresión de solicitud/respuesta habilitada y logger BASIC con DEBUG para el paquete de clientes. Se usa el prefijo vigente spring.cloud.openfeign; el prefijo feign.client del ejemplo de la consigna ya no configura estos clientes. Los destinos se resuelven por nombre mediante Eureka; no se fija una URL para cada cliente.

Referencia: [propiedades de Spring Cloud OpenFeign](https://docs.spring.io/spring-cloud-openfeign/reference/spring-cloud-openfeign.html).

Desde el sprint 13, ms-turno configura los circuitos mascotaService y veterinarioService: ventana COUNT_BASED de 10, mínimo 5 llamadas, umbral 50%, espera OPEN de 10 segundos y tres llamadas de prueba HALF_OPEN. Se habilita la transición automática a HALF_OPEN. El Circuit Breaker envuelve Retry para contar una llamada lógica y rechazar inmediatamente cuando está OPEN. Retry tiene tres intentos totales, esperas de 1 y 2 segundos y un predicado de errores transitorios; los 404 de negocio no se reintentan ni abren circuitos.

Actuator de Turnos expone health, info, metrics, circuitbreakers, circuitbreakerevents y retryevents. El detalle del circuito informa OPEN/CIRCUIT_OPEN o HALF_OPEN/CIRCUIT_HALF_OPEN; el agregador general puede conservar UP. Readiness comprueba el estado de la aplicación y su base, y se usa en Compose para distinguir degradación de dependencia de la disponibilidad del proceso. Los demás servicios mantienen su exposición histórica; los valores de env permanecen ocultos.

El RequestRateLimiter de Gateway usa Redis y limita Turnos por dirección remota de cliente: 10 tokens/segundo, capacidad 20, costo 1. La clave no depende de parámetros ni de cabeceras suministradas por el cliente. Se requiere Redis antes de iniciar Gateway; REDIS_HOST y REDIS_PORT permiten sustituir localhost:6379. Compose usa el nombre redis y mantiene su puerto dentro de la red.

Desde el sprint 14, ms-auth usa patitas_auth, BCrypt y JWT HS256 de 24 horas. ms-auth.yml y api-gateway.yml reciben jwt.secret mediante ${JWT_SECRET}; la clave real se genera en .env y se suministra solo a esos dos procesos. No se guarda una clave por defecto en este repositorio. Los demás servicios no reciben JWT_SECRET ni incorporan validación de tokens.

Las rutas /auth/** son públicas. Gateway exige Bearer válido en las demás entradas, incluyendo aliases y Actuator; permite el preflight CORS. X-Auth-User se obtiene del subject verificado y sustituye cualquier valor recibido. La configuración de ruta permanece en Git y la verificación se implementa en AuthenticationFilter reactivo. Compose comprueba el puerto TCP de Gateway porque el health HTTP también exige token.

Los cambios de configuración se aplican al iniciar el microservicio. Esta entrega no incorpora Spring Cloud Bus ni promete refresco dinámico de propiedades sin reiniciar.

Código e instrucciones: [patitas-felices-Orue-Marighetti](https://github.com/ggomarighetti/patitas-felices-Orue-Marighetti).
