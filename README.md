# Utopía — Web + Sistema de Reservas

Web de portfolio para Revcore Labs. Restaurante de alta gama ficticio ubicado en Punta Carretas, Montevideo.

**Live:** pendiente deploy en GitHub Pages

---

## Archivos

| Archivo | Descripción |
|---|---|
| `index.html` | Web completa (landing + wizard de reservas) |
| `utopia-logo.png` | Logo original del restaurante |
| `n8n-workflow-reservas.json` | Workflow n8n importable (disponibilidad + confirmación) |
| `diseno-referencia.html` | Diseño original exportado del canvas de Claude |

---

## Usar el sitio

Abrir `index.html` en cualquier browser. No requiere servidor.

El wizard de reservas funciona visualmente en modo local. Para activar el backend real (emails, Google Sheet), completar los pasos de n8n abajo.

---

## Configurar el workflow n8n

### 1. Crear el Google Sheet

Crear un Google Sheet llamado **"Utopía Reservas"** con una hoja llamada `Reservas` y estas columnas exactas en la fila 1:

```
id | nombre | email | telefono | fecha | horario | mesa | personas | solicitudes | confirmado | cancelado | cancel_token | asistio | feedback_enviado
```

### 2. Importar el workflow

1. Ir a `n8n.revcorelabs.com`
2. Crear nuevo workflow → Importar desde archivo
3. Seleccionar `n8n-workflow-reservas.json`
4. Reemplazar los 3 valores marcados con `REEMPLAZAR_CON_ID_DEL_GOOGLE_SHEET` con el ID real del Sheet (está en la URL: `docs.google.com/spreadsheets/d/ESTE_ES_EL_ID/`)
5. Conectar las credenciales: Google Sheets OAuth2 y Gmail OAuth2
6. Activar el workflow

### 3. Los workflows de Cron (crear manualmente)

El JSON incluye los webhooks (disponibilidad + reservar). Los dos flujos de email automático se crean aparte en n8n:

**Recordatorio (9:00 AM del día de la reserva):**
- Trigger: Schedule → todos los días a las 09:00
- Google Sheets Read → filtrar `fecha = hoy` + `confirmado = Sí` + `recordatorio_enviado ≠ Sí`
- Gmail Send → "Te esperamos esta noche en Utopía"
- Google Sheets Update → marcar `recordatorio_enviado = Sí`

**Feedback (2 horas después del cierre):**
- Trigger: Schedule → todos los días a las 01:00 AM
- Google Sheets Read → filtrar `fecha = ayer` + `asistio = Sí` + `feedback_enviado = No`
- Gmail Send → "¿Cómo estuvo tu experiencia?"
- Google Sheets Update → marcar `feedback_enviado = Sí`

---

## Actualizar URL del webhook

Si cambia la instancia de n8n, actualizar en `index.html` las dos constantes al inicio del script:

```js
const URL_DISPONIBILIDAD = 'https://n8n.revcorelabs.com/webhook/utopia-disponibilidad';
const URL_RESERVAR       = 'https://n8n.revcorelabs.com/webhook/utopia-reservar';
```

---

## Deploy en GitHub Pages

1. Crear repo público `utopia-restaurante` en github.com/revcorelabs
2. Subir los archivos (sin `diseno-referencia.html`)
3. Settings → Pages → Branch: main → Folder: / (root) → Save
4. URL: `https://revcorelabs.github.io/utopia-restaurante`
