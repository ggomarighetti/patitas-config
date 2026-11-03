# Configuración de Patitas Felices

Repositorio público de configuración de los sprints 10 y 11 de Microservicios y APIs Escalables.

Config Server obtiene los archivos de la rama `main`. `application.yml` contiene Eureka, Actuator y el marcador `patitas.config-source`. Los archivos `ms-dueno.yml`, `ms-mascota.yml`, `ms-veterinario.yml` y `ms-turno.yml` definen el puerto, la base de datos y la información de cada servicio.

Las credenciales se resuelven con variables de entorno en cada microservicio: `DB_USERNAME` y `DB_PASSWORD`. `DB_URL` y `EUREKA_URL` permiten sustituir las direcciones locales al ejecutar con Docker. No se almacena una contraseña real en Git.

Config Server consulta este repositorio en `http://localhost:8888/{application}/default` y responde un objeto Environment con las propiedades y el commit de origen. La representación YAML está en `http://localhost:8888/{application}-default.yml`.

Orden de arranque: Config Server (8888), Eureka Server (8761), ms-mascota (8082), ms-veterinario (8083) y ms-turno (8084). ms-dueno (8081) también está disponible. Cada servicio utiliza su propia base de datos y las direcciones de Docker se suministran mediante variables de entorno.

Desde el sprint 11, ms-turno obtiene los timeouts de OpenFeign desde ms-turno.yml: conexión de 3000 ms y lectura de 5000 ms, compresión de solicitud/respuesta habilitada y logger BASIC con DEBUG para el paquete de clientes. Se usa el prefijo vigente spring.cloud.openfeign; el prefijo feign.client del ejemplo de la consigna ya no configura estos clientes. Los destinos se resuelven por nombre mediante Eureka; no se fija una URL para cada cliente.

Referencia: [propiedades de Spring Cloud OpenFeign](https://docs.spring.io/spring-cloud-openfeign/reference/spring-cloud-openfeign.html).

Actuator expone health, info, metrics, env, beans y mappings para el entorno local de la consigna. Los valores de env se mantienen ocultos. Docker publica los servicios únicamente en la interfaz local.

Los cambios de configuración se aplican al iniciar el microservicio. Esta entrega no incorpora Spring Cloud Bus ni promete refresco dinámico de propiedades sin reiniciar.

Código e instrucciones: [patitas-felices-Orue-Marighetti](https://github.com/ggomarighetti/patitas-felices-Orue-Marighetti).
