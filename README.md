# AgroShift Michoacán · AgroSmart MX

Aplicación demostrativa para acompañar decisiones en parcelas de Michoacán. Frontend React/Vite con dashboard móvil, mapa Leaflet, gráficas Recharts, simulador de sensores y endpoints de Azure Functions Node.js v4.

## Requisitos y ejecución local

- Node.js 20 LTS o posterior.
- Para el API local: Azure Functions Core Tools v4.

```bash
npm install
npm run dev
```

El frontend funciona en modo demostración si `/api` no está disponible. Para iniciar Functions localmente desde la raíz instala también las dependencias de `api` (`npm install --prefix api`) y usa `npm run api:start`. Vite entrega el frontend en `http://localhost:5173`; para desarrollo con Functions usa SWA CLI o un proxy local dirigido a `http://localhost:7071`.

## NASA POWER y límites de las fuentes

`GET /api/nasa-telemetry` consulta NASA POWER Daily AG para T2M, PRECTOTCORR, GWETTOP y GWETROOT, con caché en memoria de seis horas y datos de respaldo explícitamente señalados. Las Functions HTTP usan `authLevel: anonymous` para el proxy integrado de Static Web Apps. El POST IoT requiere `IOT_WRITE_KEY` y encabezado `x-iot-key`; rota la clave y usa un almacén seguro de dispositivos en producción. GWETTOP/GWETROOT son variables de humedad modeladas de NASA POWER, no telemetría SMAP de paso ni mediciones de sensor. Los valores NDVI de MODIS/Landsat y elevación SRTM se devuelven como no disponibles salvo respaldo de demo; no se presentan como observaciones reales.

## Azure Static Web Apps

El flujo de GitHub Actions requiere crear el secreto `AZURE_STATIC_WEB_APPS_API_TOKEN` en el repositorio. Conserva un `package-lock.json` generado por `npm install` en el primer commit de despliegue.

```bash
az login
az group create --name rg-agroshift-mx --location centralus
az staticwebapp create --name agroshift-michoacan --resource-group rg-agroshift-mx --location centralus --source https://github.com/ORGANIZACION/REPOSITORIO --branch main --app-location "/" --api-location "api" --output-location "dist" --login-with-github
```

Si `--login-with-github` no está disponible en la versión de Azure CLI instalada, crea el recurso desde Azure Portal o instala/actualiza la extensión de Static Web Apps y configura el workflow; agrega después el token de despliegue en GitHub Actions como `AZURE_STATIC_WEB_APPS_API_TOKEN`. Reemplaza organización/repositorio por el URL real.

## Android / Capacitor

Tras preparar un toolchain Android (Android Studio/JDK/SDK):

```bash
npm install
npm run build
npx cap add android
npx cap sync android
npx cap open android
```

La configuración PWA y Capacitor permite iniciar la empaquetación; la firma de un APK de producción requiere configurar un keystore y las credenciales de firma fuera del repositorio.

## Persistencia y seguridad antes de producción

El endpoint IoT incluido usa almacenamiento en memoria para que la muestra funcione sin recursos Azure adicionales; no persiste entre reinicios ni instancias. Para producción, sustituirlo por Azure Table Storage/Cosmos DB, autenticar los endpoints (Entra ID o tokens de dispositivo), validar límites/cuotas y configurar alertas/observabilidad. Añadir en Static Web Apps solo los orígenes CORS concretos si se sirve el API fuera del proxy integrado; no se necesita CORS para frontend y API bajo el mismo origen.
