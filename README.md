
# make-pronosticos-deportivos

Este repositorio contiene un **Blueprint de Make (anteriormente Integromat)** para generar pronósticos deportivos con una confianza del 75% o más, utilizando la API de Bzzoiro Sports y enviando los resultados a un bot de Telegram.

## Características

- **Activación automática:** Se puede configurar para ejecutarse a intervalos regulares (ej. diariamente a las 9:00 AM).
- **Activación manual:** Se puede activar mediante un mensaje al bot de Telegram con palabras clave como "análisis", "pronósticos" o "picks".
- **Integración con Bzzoiro Sports API:** Obtiene partidos del día, estadísticas de equipos y datos H2H.
- **Cálculo de probabilidades:** Implementa modelos para 1X2, Over/Under 2.5 Goles, BTTS, Corners Over/Under y Doble Oportunidad (requiere configuración manual de fórmulas en Make).
- **Filtrado inteligente:** Solo reporta pronósticos con una probabilidad calculada del 75% o superior.
- **Notificaciones por Telegram:** Envía un mensaje formateado con los picks del día a un chat de Telegram.

## Requisitos previos

- Una cuenta de Make (integromat.com).
- Un bot de Telegram configurado con [@BotFather](https://t.me/BotFather).
- Un Chat ID de Telegram (puedes obtenerlo con [@userinfobot](https://t.me/userinfobot)).
- Una API Key de Bzzoiro Sports (puedes obtenerla en [sports.bzzoiro.com](https://sports.bzzoiro.com)).

## Instalación y Configuración

Sigue estos pasos para poner en marcha el escenario de Make:

### 1. Configurar el Bot de Telegram

Si aún no lo has hecho:

1.  Habla con [@BotFather](https://t.me/BotFather) en Telegram.
2.  Crea un nuevo bot y guarda el **Token del Bot**.
3.  Habla con [@userinfobot](https://t.me/userinfobot) para obtener el **Chat ID** del chat o grupo donde quieres recibir los pronósticos.

### 2. Obtener la API Key de Bzzoiro Sports

1.  Regístrate en [sports.bzzoiro.com](https://sports.bzzoiro.com).
2.  Obtén tu **API Key**.

### 3. Importar el Blueprint en Make

1.  Descarga el archivo `make_blueprint.json` de este repositorio.
2.  En tu cuenta de Make, ve a **Scenarios**.
3.  Haz clic en **Create a new scenario**.
4.  En la parte superior derecha, haz clic en el menú de tres puntos (`...`) y selecciona **Import Blueprint**.
5.  Sube el archivo `make_blueprint.json`.

### 4. Configurar Módulos en el escenario de Make

Una vez importado, deberás configurar las conexiones y los detalles de cada módulo:

#### a) Módulo "Telegram > Watch Updates" (Módulo 1)

1.  Haz clic en el módulo.
2.  Crea una nueva conexión de Telegram usando el **Token del Bot** que obtuviste de BotFather.
3.  Configura el webhook para que escuche los mensajes entrantes.

#### b) Módulos "HTTP > Make a request" (Módulos 3, 5, 6)

1.  Para cada uno de estos módulos, haz clic en él.
2.  En la sección de **Headers**, añade un nuevo header:
    *   **Name:** `Authorization`
    *   **Value:** `Token TU_API_KEY_BZZOIRO` (reemplaza `TU_API_KEY_BZZOIRO` con tu clave real de Bzzoiro Sports).

#### c) Módulo "Text Aggregator" (Módulo 7) y Lógica de Probabilidades

Este es el módulo más crítico y requerirá **configuración manual avanzada**.

-   El blueprint incluye un placeholder para la lógica de cálculo de probabilidades. **Make no permite la importación de código JavaScript complejo como n8n.**
-   Deberás recrear la lógica de cálculo de probabilidades (Poisson, 1X2, Over/Under, BTTS, Corners, Doble Oportunidad) utilizando las **herramientas de Make**:
    *   Módulos de **Tools > Set multiple variables** para almacenar resultados intermedios.
    *   Módulos de **Tools > Numeric aggregator** o **Tools > Text parser**.
    *   **Fórmulas y funciones** de Make en los campos de los módulos.
    *   Si tu plan de Make lo permite, un módulo de **Code > Run Node.js** o **Code > Run Python** podría simplificar la implementación de la función `poisson` y los cálculos complejos.
-   Asegúrate de que el resultado final de los cálculos se formatee en una cadena de texto adecuada para el mensaje de Telegram.

#### d) Módulo "Telegram > Send a Text Message or a Reply" (Módulo 8)

1.  Haz clic en el módulo.
2.  Selecciona la conexión de Telegram que creaste.
3.  En el campo **Chat ID**, introduce tu **Chat ID** de Telegram (`5566290055`).
4.  Asegúrate de que el campo **Text** contenga la salida formateada del módulo "Text Aggregator" (Módulo 7).

### 5. Activar el escenario

1.  Una vez configurado todo, activa el escenario en Make.
2.  Configura el **Schedule Setting** del módulo "Telegram > Watch Updates" para que se ejecute a la hora deseada (ej. 9:00 AM hora de la Ciudad de México).

## Pruebas

- **Activación automática:** Espera a la hora programada o ejecuta el escenario manualmente.
- **Activación manual:** Envía un mensaje a tu bot de Telegram con las palabras "análisis", "pronósticos" o "picks".

## Notas importantes

- El blueprint proporcionado es una **estructura básica**. La lógica de cálculo de probabilidades es compleja y requerirá un esfuerzo significativo para ser implementada completamente con las herramientas nativas de Make. Considera usar un módulo de código si tu plan lo permite o un servicio externo para los cálculos.
- Los endpoints de Bzzoiro Sports API utilizados en este flujo (`/football/matches/today`, `/football/teams/{teamId}/stats`, `/football/matches/{id}/h2h`) se basan en la descripción original del prompt. Si la API real de Bzzoiro Sports utiliza `/api/events/` o `/api/teams/{id}/` con una estructura diferente, es posible que necesites ajustar los módulos HTTP y los módulos de procesamiento de datos.

---

**Make Blueprint JSON:**

```json
