
# n8n-pronosticos-deportivos

Este repositorio contiene un flujo de n8n para generar pronósticos deportivos con una confianza del 75% o más, utilizando la API de Bzzoiro Sports y enviando los resultados a un bot de Telegram.

## Características

- **Activación automática:** Se ejecuta diariamente a las 9:00 AM (hora de la Ciudad de México).
- **Activación manual:** Se puede activar enviando un mensaje al bot de Telegram con palabras clave como "análisis", "pronósticos" o "picks".
- **Integración con Bzzoiro Sports API:** Obtiene partidos del día, estadísticas de equipos y datos H2H.
- **Cálculo de probabilidades:** Implementa modelos para 1X2, Over/Under 2.5 Goles, BTTS, Corners Over/Under y Doble Oportunidad.
- **Filtrado inteligente:** Solo reporta pronósticos con una probabilidad calculada del 75% o superior.
- **Notificaciones por Telegram:** Envía un mensaje formateado con los picks del día a un chat de Telegram.

## Requisitos previos

- Una instancia de n8n (self-hosted o cloud, versión >= 1.0).
- Un bot de Telegram configurado con [@BotFather](https://t.me/BotFather).
- Un Chat ID de Telegram (puedes obtenerlo con [@userinfobot](https://t.me/userinfobot)).
- Una API Key de Bzzoiro Sports (puedes obtenerla en [sports.bzzoiro.com](https://sports.bzzoiro.com)).

## Instalación y Configuración

Sigue estos pasos para poner en marcha el flujo de n8n:

### 1. Configurar el Bot de Telegram

Si aún no lo has hecho:

1.  Habla con [@BotFather](https://t.me/BotFather) en Telegram.
2.  Crea un nuevo bot y guarda el **Token del Bot**.
3.  Habla con [@userinfobot](https://t.me/userinfobot) para obtener el **Chat ID** del chat o grupo donde quieres recibir los pronósticos.

### 2. Obtener la API Key de Bzzoiro Sports

1.  Regístrate en [sports.bzzoiro.com](https://sports.bzzoiro.com).
2.  Obtén tu **API Key**.

### 3. Importar el flujo en n8n

1.  Copia el contenido del archivo `n8n_workflow.json` de este repositorio.
2.  En tu instancia de n8n, ve a **Workflows**.
3.  Haz clic en **New** y luego en **Import from JSON**.
4.  Pega el contenido del JSON y haz clic en **Import**.

### 4. Configurar Credenciales en n8n

#### a) Credencial de Telegram API

1.  En n8n, ve a **Credentials**.
2.  Haz clic en **New Credential**.
3.  Busca y selecciona **Telegram API**.
4.  En el campo **Bot Token**, pega el **Token del Bot** que obtuviste de BotFather.
5.  Guarda la credencial con un nombre descriptivo, por ejemplo: `Telegram Bot Account`.

#### b) Credencial de Bzzoiro Sports API

1.  En n8n, ve a **Credentials**.
2.  Haz clic en **New Credential**.
3.  Busca y selecciona **Header Auth**.
4.  En el campo **Name**, pon `Authorization`.
5.  En el campo **Value**, pon `Token TU_API_KEY_BZZOIRO` (reemplaza `TU_API_KEY_BZZOIRO` con tu clave real de Bzzoiro Sports).
6.  Guarda la credencial con un nombre descriptivo, por ejemplo: `Bzzoiro Sports API Key`.

### 5. Configurar Nodos en el flujo de n8n

1.  Abre el flujo importado.
2.  **Nodo "Telegram Send Message" (node16):**
    *   Haz clic en el nodo.
    *   En la sección **Credentials**, selecciona la credencial de Telegram que creaste (`Telegram Bot Account`).
    *   Asegúrate de que el campo `Chat ID` esté configurado con tu **Chat ID** de Telegram (`5566290055`).
3.  **Nodos HTTP (node5, node8, node9, node10):**
    *   Para cada uno de estos nodos, haz clic en él.
    *   En la sección **Credentials**, selecciona la credencial de Bzzoiro Sports API que creaste (`Bzzoiro Sports API Key`).

### 6. Configurar Webhook para Telegram (para activación manual)

Para que el bot de Telegram pueda activar el flujo, necesitas configurar un webhook en tu bot que apunte a la URL de tu n8n.

1.  Obtén la URL del Webhook de tu nodo "Webhook Trigger" (node2) en n8n. Estará en la configuración del nodo, algo como `https://your-n8n-instance.com/webhook/telegram-trigger`.
2.  Usa la API de Telegram para configurar el webhook. Puedes hacerlo con una solicitud HTTP simple:

    ```bash
    curl -F "url=https://your-n8n-instance.com/webhook/telegram-trigger" \
         -F "allowed_updates=message" \
         "https://api.telegram.org/bot<TU_BOT_TOKEN>/setWebhook"
    ```
    Reemplaza `<TU_BOT_TOKEN>` con el token de tu bot de Telegram y `https://your-n8n-instance.com/webhook/telegram-trigger` con la URL real de tu webhook de n8n.

### 7. Activar el flujo

1.  Una vez configurado todo, activa el flujo en n8n.

## Pruebas

- **Activación automática:** Espera hasta las 9:00 AM (hora de la Ciudad de México) o cambia el cron del nodo "Schedule Trigger" para una prueba inmediata.
- **Activación manual:** Envía un mensaje a tu bot de Telegram con las palabras "análisis", "pronósticos" o "picks".

## Notas importantes

- Los endpoints de Bzzoiro Sports API utilizados en este flujo (`/football/matches/today`, `/football/teams/{teamId}/stats`, `/football/matches/{id}/h2h`) se basan en la descripción original del prompt. Si la API real de Bzzoiro Sports utiliza `/api/events/` o `/api/teams/{id}/` con una estructura diferente, es posible que necesites ajustar los nodos HTTP y los nodos Code para parsear correctamente las respuestas.
- El nodo "Code: Calcular Probabilidades" asume que los datos de estadísticas de equipo y H2H se obtienen directamente de los endpoints especificados. Si la API de Bzzoiro no proporciona estos datos de esa manera, el código necesitará modificaciones.

---

**n8n Workflow JSON:**

```json
