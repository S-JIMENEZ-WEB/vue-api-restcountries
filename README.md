# Atlas — Explorador de países con Vue 3

Aplicación web académica que consulta REST Countries API y presenta países en tarjetas filtrables. Incluye `fetch()`, `onMounted()`, `ref()`, `computed()`, `v-for`, `v-if`, `v-model`, estados de carga y errores, y despliegue automatizado en GitHub Pages.

## 1. Ejecutar en tu computadora

Necesitas Node.js instalado. Abre una terminal dentro de esta carpeta y ejecuta:

```bash
npm install
npm run dev
```

Abre en el navegador la dirección local que indique Vite (por lo general `http://localhost:5173`). Para verificar el resultado de producción:

```bash
npm run build
npm run preview
```

## 2.1 Hacer varios commits y subirlo a tu repositorio

Crea primero un repositorio vacío en GitHub llamado `vue-api-restcountries`. Después abre una terminal en esta carpeta y ejecuta los siguientes comandos, reemplazando `TU-USUARIO` por tu usuario real de GitHub. Si Git solicita identificarte, configura tu nombre y correo con `git config --global user.name "Tu nombre"` y `git config --global user.email "tu-correo"`.

```bash
git init -b main
git add package.json index.html vite.config.js README.md .gitignore
git commit -m "chore: iniciar proyecto Vue 3"
git add src/main.js src/style.css src/App.vue
git commit -m "feat: crear explorador de países con REST Countries"
git add .github/workflows/deploy.yml
git commit -m "ci: configurar despliegue en GitHub Pages"
git remote add origin https://github.com/TU-USUARIO/vue-api-restcountries.git
git push -u origin main
```

Cuando tengas conexión a internet, ejecuta también `npm install` antes de probar el proyecto. Si se genera `package-lock.json`, inclúyelo en otro commit para registrar las versiones resueltas de las dependencias.

## 3. Publicarlo en GitHub Pages

1. En GitHub crea un repositorio público llamado exactamente `vue-api-restcountries`.
2. Sube el contenido de esta carpeta al repositorio, incluida la carpeta oculta `.github/workflows`.
3. En el repositorio abre **Settings → Pages** y selecciona **GitHub Actions** como fuente de publicación.
4. Entra a **Actions** y espera a que termine en verde el flujo “Publicar Vue en GitHub Pages”.
5. El sitio tendrá una dirección similar a `https://TU-USUARIO.github.io/vue-api-restcountries/`. Reemplaza `TU-USUARIO` por tu nombre de usuario real.
6. Envía por separado el enlace del repositorio y el enlace del sitio publicado.

La configuración de `vite.config.js` ya usa `/vue-api-restcountries/` como ruta base. Si cambias el nombre del repositorio, actualiza esa ruta.

## 4. API y datos

La app utiliza el endpoint de listado de REST Countries v5 con `limit=100` y `offset` para recuperar páginas, después filtra en el navegador por nombre o región. El enlace del enunciado con `q=Canada` es una búsqueda específica; por eso, para poder explorar diferentes países, esta versión usa el endpoint de listado del mismo servicio.

Se utiliza `rc_live_demo` porque es la clave de demostración del enunciado/documentación. Si el acceso desde el navegador produce un error de autorización o CORS, crea una clave en REST Countries y autoriza el hostname donde se publicará el sitio. No publiques claves privadas en un repositorio público.

Documentación: https://restcountries.com/docs/countries

## 5. Evidencias de cumplimiento

Usa el PDF de plantilla incluido por separado. En el navegador toma capturas reales del sitio; en VS Code toma capturas nítidas del archivo `src/App.vue`. Respeta la indicación docente de **una captura por página** y coloca una explicación breve en cada evidencia.

Archivos importantes:

- `src/App.vue`: lógica Vue, `fetch()`, `onMounted()`, estado reactivo, filtros y renderizado.
- `src/style.css`: diseño adaptable.
- `vite.config.js`: ruta base de GitHub Pages.
- `.github/workflows/deploy.yml`: construcción y despliegue automáticos.
