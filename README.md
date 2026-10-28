# Ponches Web — instalar y activar

Este proyecto conserva la app de Ponches y añade una PWA, inicio de sesión con Google y sincronización central en el Drive del administrador. No contiene tus ponches reales, credenciales ni cuentas de ejemplo activadas.

**Estado:** código preparado y comprobado localmente. Todavía no está desplegado en tu Netlify ni conectado a tu Google. El acceso real, la autorización de Drive y la prueba entre dos celulares requieren tu configuración.

## 1. Sube el proyecto a GitHub

1. Descomprime `Ponches_Web_Netlify.zip`.
2. Crea un repositorio **privado** en GitHub, por ejemplo `ponches-web`.
3. Sube el contenido de la carpeta `ponches-web`: `public`, `netlify`, `server`, `scripts`, `tests`, `package.json`, `package-lock.json`, `netlify.toml` y este archivo. Los archivos deben estar en la raíz del repositorio, sin otra carpeta contenedora.
4. Incluye `.gitignore` si tu selector permite ver archivos ocultos. No subas un `.env` con valores reales. `.env.example` solo contiene ejemplos.

El ZIP es **un proyecto para desplegar con funciones de servidor**, no el antiguo ZIP para convertir en APK. No lo subas a Netlify Drop: necesitamos que Netlify instale dependencias y despliegue las funciones desde GitHub.

## 2. Publica en Netlify

1. En Netlify: Add new project → Import an existing project → GitHub.
2. Selecciona el repositorio privado.
3. Netlify leerá `netlify.toml`. Comprueba:
   - Base directory: vacío.
   - Build command: `npm run build`.
   - Publish directory: `public`.
   - Functions directory: `netlify/functions`.
4. Despliega. La app puede abrirse sin cuentas mientras completas la configuración.
5. Escoge el nombre definitivo del sitio. Usaremos como ejemplo `https://TU-SITIO.netlify.app`. Sustitúyelo por el enlace real en TODOS los pasos siguientes.

## 3. Configura Google

En tu proyecto de Google Cloud:

1. Habilita **Google Drive API**.
2. En Google Auth Platform, configura nombre Ponches, correo de asistencia y correo de contacto. Público: Externo para cuentas Gmail personales. Añade tu correo y otra cuenta a usuarios de prueba.
3. En Data Access añade `openid`, `email`, `profile` y `https://www.googleapis.com/auth/drive.file`.
4. Crea un cliente OAuth de tipo **Aplicación web**.
5. Origen JavaScript autorizado: `https://TU-SITIO.netlify.app`.
6. URI de redirección autorizada, exactamente:

   ```text
   https://TU-SITIO.netlify.app/api/auth/callback
   ```

7. Guarda el ID de cliente y el secreto. El secreto va solo en Netlify.
8. Para la información pública de la aplicación puedes usar:
   - Página principal: `https://TU-SITIO.netlify.app/`
   - Privacidad: `https://TU-SITIO.netlify.app/privacidad.html`
   - Ayuda: `https://TU-SITIO.netlify.app/ayuda.html`

La pantalla de Google pedirá solo identidad a los usuarios normales. El permiso de Drive se solicita únicamente al administrador al pulsar **Conectar mi Drive**.

## 4. Añade las variables privadas en Netlify

Project configuration → Environment variables. Añade estas variables para el entorno Production con alcance **Functions**. Si tu plan no permite elegir alcance, usa el alcance general disponible. Marca los secretos como privados donde se ofrezca esa opción.

| Variable | Valor |
| --- | --- |
| `APP_URL` | Tu dirección HTTPS, sin `/` al final. |
| `GOOGLE_CLIENT_ID` | ID que termina en `.apps.googleusercontent.com`. |
| `GOOGLE_CLIENT_SECRET` | Secreto del cliente Google. |
| `ADMIN_EMAIL` | Tu correo de Google, el dueño del Drive central. |
| `TOKEN_ENCRYPTION_KEY` | Una clave aleatoria de 32 bytes en base64. |
| `ALLOWED_EMAILS` | Para empezar: tu correo y el de prueba, separados por coma. |

Para generar `TOKEN_ENCRYPTION_KEY`, ejecuta en una computadora con Node instalado o en la terminal de Google Cloud Shell:

```sh
node -e "console.log(require('crypto').randomBytes(32).toString('base64'))"
```

Copia el resultado directamente en Netlify. No lo publiques ni lo introduzcas en el HTML. Conserva esta clave: cambiarla obliga a recuperar o volver a configurar la autorización cifrada de Drive. La variable `ALLOWED_EMAILS` limita las cuentas aceptadas; dejarla vacía permite el acceso con cualquier cuenta de Google. Mantén la lista restringida durante las pruebas.

Después de guardar las variables, ejecuta **Trigger deploy → Deploy project** para que las funciones reciban la configuración.

## 5. Conecta tu Drive

1. Abre la web → Ajustes → Cuenta.
2. Pulsa **Continuar con Google** e inicia sesión con el correo que pusiste en `ADMIN_EMAIL`.
3. Pulsa **Conectar mi Drive** y autoriza la misma cuenta.
4. Se creará una carpeta llamada **Ponches · Datos privados** en tu Drive. No la compartas con los demás usuarios.
5. El estado debe confirmar la conexión. Crea un ponche de prueba y comprueba que aparezca **Sincronizado**.

El servidor conserva cifrada la autorización persistente. Los usuarios no reciben esa autorización ni permisos sobre la carpeta. El servidor verifica la sesión y selecciona los archivos según el identificador de Google de cada cuenta.

## 6. Prueba dos cuentas y dos dispositivos

1. Abre la web con tu cuenta en dos navegadores o celulares. Añade un ponche en uno, espera a que sincronice y abre el otro. Comprueba las mismas fechas y horas.
2. Entra con la segunda cuenta permitida: debe comenzar sin tus registros.
3. Haz cambios sin conexión, reconecta y verifica la sincronización.
4. Modifica la misma cuenta desde dos dispositivos antes de sincronizar: debe pedir revisar las versiones. Descarga ambas antes de elegir.
5. Prueba descargar JSON, importar un respaldo anterior, cargar el PDF de un mes y descargar la nómina. En iPhone, el enlace o **Guardar o compartir** permite abrir la hoja de compartir del sistema.
6. Prueba cerrar sesión. La copia local por cuenta se conserva, pero ya no puede sincronizar con la sesión cerrada.

En el modo Testing de Google, la autorización de Drive puede caducar a los siete días. Antes de usarla permanentemente, cambia el proyecto OAuth a producción y completa los requisitos que muestre Google. Las cuentas deben estar habilitadas también por `ALLOWED_EMAILS` mientras mantengas esa variable restringida.

## 7. Añade Ponches al inicio

- Android: Chrome → menú → Instalar aplicación o Añadir a pantalla de inicio.
- iPhone: Safari → Compartir → Añadir a pantalla de inicio.

Abre la web con conexión y deja que termine su primera carga. La PWA guarda los recursos necesarios, incluidas las herramientas PDF. Los cambios se sincronizan con la app abierta y conexión disponible; no hay sincronización en segundo plano con la app cerrada.

## Datos anteriores y límites de esta versión

- El formato JSON anterior sigue siendo compatible. Importa desde Ajustes → Datos.
- Los datos de un APK no pasan automáticamente a la web. Conserva ese APK y sus datos hasta recuperarlos.
- Si hay registros sin cuenta en esta misma dirección, puedes importarlos explícitamente desde Ajustes → Cuenta. Nunca se asignan automáticamente a la primera cuenta que entre.
- Se sincronizan **ponches, ajustes de horas, horarios, tarifa y preferencias de nómina**. El tema visual es una preferencia local por dispositivo.
- Los formularios PDF se guardan en el navegador y por cuenta. Se incluyen en el respaldo JSON, pero no se sincronizan automáticamente a otros celulares.
- El Drive central contiene los archivos JSON de todos los usuarios. **Tú, como dueño, puedes acceder a ellos**. La aplicación informa de esto antes de sincronizar.
- Netlify Blobs guarda únicamente la configuración cifrada de Drive, sesiones y referencias/versiones de archivos. Es necesario además de Drive para las comprobaciones de concurrencia; no es una segunda copia de los ponches.
- Las sesiones duran hasta siete días. La autorización de Drive puede revocarse o caducar; en ese caso se conservan los cambios locales y el administrador debe reconectar.
- Máximo de 2 MB de JSON por sincronización de registros. Esto no limita el peso de la aplicación ni los PDF locales. Los PDFs individuales conservan el límite previo de 20 MB.
- Drive conserva la versión actual y una anterior por usuario; las más antiguas pasan a la papelera. Una interrupción ambigua puede dejar un archivo adicional para recuperación. No borres a mano la carpeta ni las referencias de Netlify durante el uso normal.
- No se han cambiado las reglas de nómina: los ponches originales se conservan y el PDF redondea a la media hora más cercana.

## Desarrollo y mantenimiento

```sh
npm ci
npm run build
npm test
```

Para una prueba completa de las funciones en tu equipo, usa Netlify CLI con el proyecto vinculado y sus variables configuradas. El acceso OAuth requiere una URL HTTPS autorizada. No pongas credenciales reales en pruebas ni en repositorios.

Los recursos PDF se generan al compilar en `public/vendor`; esa carpeta no se incluye en el ZIP de código ni en Git. No borres `package-lock.json`: fija las dependencias de la compilación. El código del service worker está en `scripts/sw-template.js`; `public/sw.js` se regenera al compilar.

Después de cambiar la app, sube el código al mismo repositorio. Netlify redepliega; la PWA ofrece **Actualizar Ponches** cuando detecta una nueva versión. Mantén la misma URL para conservar el almacenamiento local.

La clave de cifrado y los datos de Netlify Blobs forman parte de la recuperación del servicio. Guarda las variables privadas de forma segura. Un Drive que contenga archivos, sin sus referencias/versiones de Netlify, requiere una recuperación administrativa antes de reanudar la sincronización.

## Verificación realizada

- Pruebas automatizadas de aislamiento entre cuentas, nonce y correo verificado, control de administrador, origen de escritura, cifrado autenticado, conflictos de versiones y concurrencia.
- Pruebas del cliente de migración sin pérdida, separación de cuentas, sesión vencida, errores de red y rechazo de ediciones desde pestañas antiguas.
- Comprobación del redondeo existente sin modificar los ponches originales.
- Prueba visual en Chrome: añadir un ponche, conservarlo tras recargar y descargar un JSON con nombre y extensión automáticos, y revisar la sección Cuenta en modo oscuro.
- Pendiente: OAuth y Drive reales en tu proyecto, despliegue de Netlify, instalación/offline en celulares reales y descarga PDF en iPhone/Android.

## Referencias de configuración

- Netlify Functions: https://docs.netlify.com/build/functions/get-started/
- Variables de funciones: https://docs.netlify.com/build/functions/environment-variables/
- Netlify Blobs y escrituras condicionales: https://docs.netlify.com/build/data-and-storage/netlify-blobs/
- Google OpenID Connect: https://developers.google.com/identity/openid-connect/openid-connect
- Google OAuth de servidor: https://developers.google.com/identity/protocols/oauth2/web-server
- Permisos de Drive: https://developers.google.com/workspace/drive/api/guides/api-specific-auth
