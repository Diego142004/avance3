# Medinova — Sistema de consultas médicas | Avance 3

Proyecto académico cliente-servidor. Backend Spring Boot 3.4.5 / Java 21; frontend Angular 19 / TypeScript, formularios reactivos y SASS; base de datos MySQL 8. API REST con autenticación Bearer de sesión opaca (no JWT), Spring Security y BCrypt.

## 1. Requisitos
- Java JDK 21 (`java -version`)
- Maven 3.9+ (`mvn -v`) o Maven integrado de IntelliJ IDEA
- Node.js 22 LTS y npm (`node -v`, `npm -v`)
- MySQL 8, servicio iniciado (MySQL Workbench opcional)
- Acceso a Internet para descargar dependencias en el primer inicio.

## 2. Crear la base de datos
En MySQL Workbench, abre la conexión local con `root`, abre el archivo `database/clinica_medica.sql` y ejecuta todo con el rayo. También basta ejecutar:

```sql
CREATE DATABASE IF NOT EXISTS clinica_medica CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

Hibernate crea/actualiza las tablas al arrancar. Se inicializan automáticamente tres cuentas de prueba y registros ficticios. El SQL incluye el esquema completo; no es un volcado de una instancia ejecutada.

## 3. Backend (PowerShell)
Abre una terminal en la carpeta raíz del proyecto:

```powershell
cd backend
$env:DB_USER="root"
$env:DB_PASSWORD="TU_CONTRASEÑA_MYSQL"
mvn spring-boot:run
```

Si tu MySQL no tiene contraseña, elimina la variable con `Remove-Item Env:DB_PASSWORD -ErrorAction SilentlyContinue` (el valor por defecto es vacío). Si usas IntelliJ, puedes abrir `backend/pom.xml`, cargar el proyecto Maven y ejecutar `ClinicaApplication.java`. Configura `DB_PASSWORD` en la configuración de ejecución si corresponde, o edita `spring.datasource.password` en `backend/src/main/resources/application.properties` **solo localmente**, sin compartir credenciales personales.

Puerto de backend: `8080`. Si cambias el puerto, actualiza `API` en `frontend/src/app/core/api.ts`. El origen CORS predeterminado es `http://localhost:4200`; si usas otro, configura `FRONTEND_ORIGIN`.

## 4. Frontend (otra terminal)
Desde la carpeta raíz:

```powershell
cd frontend
npm install
npm start
```

Abre **http://localhost:4200** en el navegador. No abras `index.html` directamente. Mantén ambas terminales y MySQL activos.

## 5. Cuentas de demostración
| Usuario | Contraseña inicial | Permisos |
|---|---|---|
| admin | Clinica123! | Acceso total a los 4 CRUD y dashboard |
| medico | Clinica123! | Lectura de pacientes y médicos; CRUD citas y fórmulas |
| usuario | Clinica123! | Solo lectura de fórmulas y su métrica |

Se resolvió la diferencia de roles de la consigna de Avance 2 implementando los tres. Los permisos de MEDICO son una decisión de diseño académico (no se implementa asignación individual médico-cuenta). Las cuentas se crean solo si no existen. `DEMO_PASSWORD` permite elegir la clave inicial antes del primer arranque. Para deshabilitar datos de prueba, configura `DEMO_ENABLED=false`; entonces debes aprovisionar cuentas por un medio seguro antes de iniciar. No utilizar datos reales ni desplegar esta configuración de demostración en producción. El token expira a las 8 horas y se invalida al cerrar sesión o reiniciar backend. Las contraseñas se almacenan como hash BCrypt. CORS restringido al origen de Angular, y CSRF deshabilitado para solicitudes autenticadas exclusivamente con cabecera Bearer (no cookies).

## 6. Demostración para exposición
1. Entra como `admin`. Dashboard: cuenta pacientes, médicos, citas y fórmulas desde MySQL.
2. En Pacientes crea un registro con DNI de 8 dígitos, edítalo y prueba un DNI repetido.
3. En Médicos registra un especialista con CMP numérico de 4–10 dígitos y edítalo.
4. En Citas crea una consulta para mañana a las 10:00. Selecciona paciente y médico. Intenta registrar el mismo médico a las 10:00: se debe rechazar con HTTP 409. Cambia la cita a CANCELADA o ATENDIDA. Se puede reprogramar una cita PROGRAMADA. Los turnos son de 30 minutos y se registran en hora de Perú.
5. En Fórmulas registra un ejemplo ficticio con nombre, forma farmacéutica, concentración, composición e indicaciones documentadas; edita y elimina. El módulo es un catálogo académico, no recomienda tratamientos.
6. Cierra sesión y entra como `medico`. Revisa las acciones permitidas y el acceso de solo lectura a pacientes/médicos.
7. Entra como `usuario`. Solo debes ver fórmulas y su métrica. Intenta abrir `/pacientes`: el guard debe redirigirte al dashboard. El backend también rechaza peticiones no autorizadas.
8. Abre DevTools > Network y selecciona Fetch/XHR. Muestra GET `/api/dashboard`, POST `/api/citas`, PATCH `/api/citas/{id}/estado` y los códigos HTTP. No muestres tokens ni contraseñas en capturas.

## 7. Pruebas
Desde `backend`: `mvn test` (pruebas de integración con H2, sin MySQL). Desde `frontend`: `npm run build` (compilación Angular). Luego prueba el flujo con MySQL local.

**Estado de verificación:** se entregan fuentes y pruebas de integración escritas. Este entorno no tiene Maven ni MySQL y no puede descargar dependencias npm/Maven, por lo que no se realizó una compilación completa ni ejecución real de la aplicación. Las capturas de funcionamiento no se simulan; deben tomarse en tu equipo después de iniciar el proyecto. Ver `docs/GUIA_DE_EVALUACION.md`.

## 8. Soluciones frecuentes
- `Access denied for user root`: revisa la contraseña real de MySQL; no es la del login de Medinova.
- `Communications link failure`: inicia MySQL, comprueba el puerto (3306 por defecto).
- `mvn no se reconoce`: instala Maven y agrega su carpeta `bin` al PATH, o utiliza IntelliJ.
- Error HTTP 0 / no conecta con servidor: confirma que Spring Boot está corriendo en 8080 y accede al frontend mediante `localhost:4200`.
- Error 401: vuelve a iniciar sesión; reiniciar el backend revoca sesiones previas.
