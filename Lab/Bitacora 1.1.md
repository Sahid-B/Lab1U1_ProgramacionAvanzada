Bitácora - Laboratorio 1.1

Estudiante: Sahid Bosquez
Asignatura: Metodologías de Programación / Desarrollo de Software
Docente: John Javier Cruz Garzón
Práctica N°: 1.1 - Aplicación básica con Spring Boot, endpoints REST, Actuator, perfiles y DevTools
Fecha: 16/10/2026

==================================================
ENTORNO
==================================================
Java y Maven:
- java -version: 21.0.12
- .\mvnw.cmd -v:
  Apache Maven 3.10.0 (c43a36b8d67be7e0805a411bc0898af1a51f5472)
  Maven home: C:\Users\ESPE-SD\.m2\wrapper\dists\apache-maven-3.10.0\2aaba761688b1a6ea5a46e7e5f4d5406ed7a75ee8da95e5e3079863a23461c9b
  Java version: 25.0.2, vendor: Oracle Corporation, runtime: C:\Program Files\Java\jdk-25.0.2
  Default locale: es_EC, platform encoding: UTF-8, time zone: America/Guayaquil
  OS name: "windows 11", version: "10.0", arch: "amd64", family: "windows"
- Spring Boot 4.1.1 (la guía indica 3.x; en el asistente de proyectos de IntelliJ solo había versiones 4.x estables).
- Dependencias: Spring Web, Spring Boot Actuator, Spring Boot DevTools.
- Compilación: .\mvnw.cmd clean package -DskipTests -> BUILD SUCCESS
- Observación 1: java -version muestra Java 21, pero Maven y la ejecución de la aplicación usan el JDK 25.0.2 definido en JAVA_HOME. El proyecto se compila con release 21 (maven-compiler), por lo que es compatible con Java 21.
- Observación 2: el paquete del proyecto quedó como com.espe.Lab (con L mayúscula). No afecta el funcionamiento.

==================================================
VERIFICACIÓN BÁSICA (perfil por defecto, puerto 8080)
==================================================
Comando: .\mvnw.cmd spring-boot:run
- curl http://localhost:8080/api/saludo
  {"mensaje":"Aplicación Spring Boot en funcionamiento - ESPE","perfil":"default"}
- curl http://localhost:8080/actuator/health
  {"groups":["liveness","readiness"],"status":"UP"}
- curl http://localhost:8080/actuator/info
  {"app":{"perfil":"default"}}
- Log: "Tomcat started on port 8080 (http)" y "Started LabApplication in 3.02 seconds"
- Nota: en la consola cmd de Windows el acento se mostró como "AplicaciÃ³n". Es un problema de codificación de la consola, no de la aplicación.
- Nota: /actuator/health incluye "groups" (liveness, readiness), propio de Spring Boot 4; la guía solo muestra {"status":"UP"}.

==================================================
REINICIO CON DEVTOOLS
==================================================
Cambio realizado:
En application.properties, la propiedad app.mensaje se cambió de
"Aplicación Spring Boot en funcionamiento - ESPE" a
"Aplicación Spring Boot con DevTools - ESPE".
Con la aplicación en ejecución se guardó el archivo y se ejecutó .\mvnw.cmd compile (BUILD SUCCESS).

Evidencia del reinicio (registro):
2026-10-16T11:47:17.088-05:00  INFO 17704 --- [lab] [  restartedMain] com.espe.Lab.LabApplication : Starting LabApplication using Java 25.0.2 with PID 17704 (C:\Users\ESPE-SD\Downloads\Lab\Lab\target\classes started by ESPE-SD in C:\Users\ESPE-SD\Downloads\Lab\Lab)
2026-10-16T11:47:17.091-05:00  INFO 17704 --- [lab] [  restartedMain] com.espe.Lab.LabApplication : No active profile set, falling back to 1 default profile: "default"
2026-10-16T11:47:17.169-05:00  INFO 17704 --- [lab] [  restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : Devtools property defaults active! Set 'spring.devtools.add-properties' to 'false' to disable
...
[restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer : Tomcat initialized with port 8080 (http)
[restartedMain] com.espe.Lab.LabApplication : Started LabApplication in 3.515 seconds



Respuesta antes y después:
- Antes:   {"mensaje":"Aplicación Spring Boot en funcionamiento - ESPE","perfil":"default"}
- Después: {"mensaje":"Aplicación Spring Boot con DevTools - ESPE","perfil":"default"}
La aplicación no se detuvo manualmente: DevTools realizó el reinicio automático.

==================================================
PERFIL DEV (puerto 8081)
==================================================
Se creó src/main/resources/application-dev.properties con:
  server.port=8081
  app.mensaje=Aplicacion Spring Boot en entorno de desarrollo - ESPE
  info.app.perfil=dev

Comando: .\mvnw.cmd spring-boot:run "-Dspring-boot.run.profiles=dev"
- curl http://localhost:8081/api/saludo
  {"mensaje":"Aplicacion Spring Boot en entorno de desarrollo - ESPE","perfil":"dev"}
- curl http://localhost:8081/actuator/health
  {"groups":["liveness","readiness"],"status":"UP"}
- curl http://localhost:8080/api/saludo
  curl: (7) Failed to connect to localhost:8080 after 2216 ms: Could not connect to server
  (el puerto 8080 ya no responde, como se esperaba)

Log del perfil dev:
2026-10-16T12:01:02.191-05:00  INFO 12416 --- [lab] [  restartedMain] com.espe.Lab.LabApplication : The following 1 profile is active: "dev"


2026-10-16T12:01:04.637-05:00  INFO 12416 --- [lab] [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer : Tomcat started on port 8081 (http) with context path '/'
2026-10-16T12:01:04.644-05:00  INFO 12416 --- [lab] [  restartedMain] com.espe.Lab.LabApplication : Started LabApplication in 2.882 seconds (process running for 3.323)

==================================================
INCIDENCIA 1
==================================================
Síntoma observado:
Al iniciar una segunda instancia de la aplicación con .\mvnw.cmd spring-boot:run mientras la primera seguía en ejecución, la segunda no arrancó y el proceso terminó con error.

Mensaje de error:
APPLICATION FAILED TO START
Description: Web server failed to start. Port 8080 was already in use.
Action: Identify and stop the process that's listening on port 8080 or configure this application to listen on another port.

Causa identificada:
La primera instancia ya estaba escuchando en el puerto 8080 (perfil por defecto). Dos procesos no pueden usar el mismo puerto al mismo tiempo, por lo que Tomcat de la segunda instancia no pudo iniciar.

Solución aplicada:
Se ejecutó la segunda instancia con el perfil dev, que cambia el puerto a 8081:
.\mvnw.cmd spring-boot:run "-Dspring-boot.run.profiles=dev"
(La alternativa era detener la primera instancia con Ctrl+C.)

Verificación posterior:
La segunda instancia arrancó correctamente en el puerto 8081 y
curl http://localhost:8081/api/saludo respondió {"mensaje":"...","perfil":"dev"}.
La primera instancia siguió disponible en el puerto 8080.

==================================================
CONCLUSIONES TÉCNICAS
==================================================
- Spring Boot permite crear un servicio REST funcional con muy poca configuración usando Spring Web.
- Actuator expone el estado (/actuator/health) y la información (/actuator/info) de la aplicación; se expusieron solo health e info, como recomienda la guía.
- Los perfiles permiten cambiar puerto y mensajes sin tocar el código: application-dev.properties sobrescribe los valores de application.properties.
- DevTools reinicia la aplicación automáticamente al recompilar, lo que acelera el desarrollo.
- Dos instancias no pueden compartir puerto; se resuelve deteniendo una o usando otro perfil/puerto.
- Se usó Spring Boot 4.1.1 en lugar de 3.x; el código y la configuración de la guía funcionaron sin cambios.

==================================================
USO DE INTELIGENCIA ARTIFICIAL
==================================================
Herramienta: Claude (Anthropic).
Consulta: orientación paso a paso para realizar el laboratorio 1.1 y ayuda para redactar esta bitácora.
Verificación realizada: se ejecutó cada paso en el equipo y se comprobó el resultado real con curl y con los registros de consola (BUILD SUCCESS, puertos 8080/8081, perfil dev, error de puerto ocupado).