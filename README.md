# Configuración de Patitas Felices

Repositorio público de configuración del sprint 10 de Microservicios y APIs Escalables.

Config Server obtiene los archivos de la rama `main`. `application.yml` contiene Eureka, Actuator y el marcador `patitas.config-source`. Los archivos `ms-dueno.yml`, `ms-mascota.yml`, `ms-veterinario.yml` y `ms-turno.yml` definen el puerto, la base de datos y la información de cada servicio.

Las credenciales se resuelven con variables de entorno en cada microservicio: `DB_USERNAME` y `DB_PASSWORD`. `DB_URL` y `EUREKA_URL` permiten sustituir las direcciones locales al ejecutar con Docker. No se almacena una contraseña real en Git.

Config Server consulta este repositorio en `http://localhost:8888/{application}/default` y responde un objeto Environment con las propiedades y el commit de origen. La representación YAML está en `http://localhost:8888/{application}-default.yml`.

Orden de arranque: Config Server (8888), Eureka Server (8761), microservicios. En esta entrega se ejecuta ms-dueno (8081); los demás contextos tienen su estructura preparada en el repositorio de código.

Actuator expone health, info, metrics, env, beans y mappings para el entorno local de la consigna. Los valores de env se mantienen ocultos. Docker publica los servicios únicamente en la interfaz local.

Los cambios de configuración se aplican al iniciar el microservicio. Esta entrega no incorpora Spring Cloud Bus ni promete refresco dinámico de propiedades sin reiniciar.

Código e instrucciones: [patitas-felices-Orue-Marighetti](https://github.com/ggomarighetti/patitas-felices-Orue-Marighetti).
