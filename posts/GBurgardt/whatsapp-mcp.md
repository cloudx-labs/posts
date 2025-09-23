# WhatsApp + MCP: transcripción automática de audios

MCP (Model Context Protocol) puede parecer complejo hasta que construyes algo real con él. Por eso vamos a llevarlo a la práctica con algo util: exponer tus audios de WhatsApp con tu propio servidor MCP para transcribir audios.

## Que es MCP?

MCP es un estándar de conexión que conecta agentes de IA con sistemas externos.

Tiene un servidor y un cliente y tienen dos formas distintas de comunicarse entre si:

- stdio (stdin/stdout): es el mecanismo estándar de Unix por el que un proceso recibe/entrega datos del entorno o de otro proceso (entrada).
- Server-Sent Events (SSE): mecanismo sobre HTTP donde el servidor mantiene la conexión abierta y envía eventos al cliente de forma continua (unidireccional).

![STDIN/STDOUT vs SSE en MCP](./assets/mcp_whatsapp_2.png)

## Arquitectura de un MCP

- Host: Claude Desktop / Cursor / Cualquier agente IA. Coordina el LLM, arranca clientes MCP y muestra resultados.
- Cliente MCP: implementación embebida en el host que se conecta a tu servidor. habla el protocolo, abre/gestiona la conexión y envía/recibe requests.
- Servidor MCP: tu programa que expone tools/resources. ejecuta acciones y devuelve datos/eventos al cliente.

El servidor expone tres cosas:

- **Tools**: Acciones que puede ejecutar (transcribir audio, buscar archivos)

- **Resources**: Datos que puede proveer (lista de contactos, archivos recientes)

- **Prompts**: Plantillas de interacción (no las usamos en este ejemplo)

![Arquitectura MCP: Host → Cliente MCP → Servidor MCP](./assets/mcp_whatsapp_1.png)

## Creando el MCP de WhatsApp

WhatsApp Desktop en macOS guarda todo localmente. Base de datos SQLite con los chats, carpetas con los archivos multimedia.

Nuestro servidor MCP va a:

1. Leer la base de datos de WhatsApp

2. Buscar archivos de audio por contacto

3. Transcribirlos usando OpenAI Whisper (SDK)

4. Devolver el texto a Claude

El código funcional está en el repositorio: [link pendiente](REPO_URL_PENDIENTE). Vamos a explicar las partes clave.

### La conexión STDIN/STDOUT

```typescript
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const transport = new StdioServerTransport();
await this.server.connect(transport);
```

Con eso el servidor escucha toda request de un cliente en STDIN y le responde por STDOUT.

Elegimos stdio porque este MCP Server corre localmente. Es el transporte más simple y estable en desktop/CLI: no abre puertos ni depende de HTTP, evita CORS/firewalls y los hosts (Claude Desktop/Cursor) lo soportan nativamente. SSE tiene sentido cuando el servidor vive remoto detrás de HTTP.

### Exponiendo capacidades

```typescript
this.server = new Server(
  {
    name: "whatsapp-audio-mcp",
    version: "1.0.0",
  },
  {
    capabilities: {
      resources: {}, // Vamos a exponer datos
      tools: {}, // Vamos a exponer acciones
    },
  }
);
```

### Resources: Los datos que exponemos

```typescript
this.server.setRequestHandler(ListResourcesRequestSchema, async () => {
  return {
    resources: [
      {
        uri: "whatsapp://contacts",
        name: "WhatsApp Contacts",
        description: "List of WhatsApp contacts with audio messages",
        mimeType: "application/json",
      },
      {
        uri: "whatsapp://recent/all",
        name: "Recent Audios",
        description: "Recent audio messages from all contacts",
        mimeType: "application/json",
      },
    ],
  };
});
```

Cada resource tiene una URI única. Cuando Claude pide `whatsapp://contacts`, nuestro servidor sabe qué devolver.

### Tools: Las acciones disponibles

```typescript
{
  name: 'transcribeAudio',
  description: 'Transcribe an audio file using OpenAI Whisper (SDK)',
  inputSchema: {
    type: 'object',
    properties: {
      audioPath: {
        type: 'string',
        description: 'Path to the audio file',
      },
    },
    required: ['audioPath'],
  },
}
```

El schema es JSON Schema estándar. Con esto Claude sabe qué parámetros enviar.

## El motor: Accediendo a WhatsApp

WhatsApp Desktop guarda todo en rutas predecibles:

```typescript
this.dbPath = path.join(
  homeDir,
  "Library/Group Containers/group.net.whatsapp.WhatsApp.shared/ChatStorage.sqlite"
);
this.mediaPath = path.join(
  homeDir,
  "Library/Group Containers/group.net.whatsapp.WhatsApp.shared/Message/Media"
);
```

La base de datos es SQLite:

```typescript
const query = `
  SELECT DISTINCT 
    ZCONTACTJID as jid,
    ZPARTNERNAME as name,
    ZLASTMESSAGEDATE as lastMessageDate
  FROM ZWACHATSESSION
  WHERE ZPARTNERNAME IS NOT NULL
  AND ZCONTACTJID NOT LIKE '%@g.us'  -- Excluir grupos
`;
```

Los archivos de audio están organizados por contacto. Escaneamos recursivamente:

```typescript
const audioExtensions = [".opus", ".m4a", ".mp3", ".aac", ".wav"];

async function scanDirectory(dir: string): Promise<void> {
  const entries = await fs.readdir(dir, { withFileTypes: true });

  for (const entry of entries) {
    if (audioExtensions.some((ext) => entry.name.endsWith(ext))) {
      // Encontramos un audio
      audioFiles.push({
        path: fullPath,
        filename: entry.name,
        modifiedDate: stats.mtime.toISOString(),
      });
    }
  }
}
```

## La transcripción: FFmpeg + OpenAI Whisper (SDK)

WhatsApp usa formato Opus, pero OpenAI Whisper prefiere MP3. Usamos FFmpeg:

```typescript
const ffmpeg = spawn("ffmpeg", [
  "-i",
  inputPath, // Audio Opus de WhatsApp
  "-acodec",
  "mp3",
  "-b:a",
  "128k",
  outputPath, // MP3 temporal
]);
```

Luego transcribimos con OpenAI Whisper (SDK):

```typescript
import OpenAI from "openai";
import fs from "node:fs";

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

const transcription = await openai.audio.transcriptions.create({
  file: fs.createReadStream(outputPath), // MP3 temporal
  model: "whisper-1",
});

const transcriptionText = transcription.text;
```

## Configurando Claude (el cliente)

En la configuración de Claude Desktop agregamos:

```json
{
  "mcpServers": {
    "whatsapp": {
      "command": "node",
      "args": ["/path/to/mcp-whatsapp-whisper/dist/server.js"]
    }
  }
}
```

Claude ahora puede invocar nuestro servidor cuando lo necesite.

## El MCP en acción

Usuario pregunta a Claude:

>

"¿Qué me dijo Lucas en el último audio que me mandó?"

Claude automáticamente:

1. Invoca `getRecentAudio(contactName: "lucas")`

2. Recibe la ruta del archivo de audio

3. Invoca `transcribeAudio(audioPath: "/path/to/audio.opus")`

4. Recibe la transcripción

5. Resume o muestra el texto completo

La transcripción se realiza en OpenAI vía API, el MP3 temporal se envía para obtener el texto. Claude orquesta; tu servidor prepara el archivo y hace la llamada.

![Claude + MCP WhatsApp: ejemplo de transcripción de audio](./assets/mcp_whatsapp_3.png)

## Las limitaciones: macOS only

Este servidor es macOS only. Las rutas de WhatsApp son específicas de Mac.

Depende de:

- WhatsApp Desktop instalado

- FFmpeg (`brew install ffmpeg`)

- OpenAI SDK (`npm i openai`) y `OPENAI_API_KEY` configurada

- Conexión a Internet

Tampoco implementamos Prompts ni Resource Templates.

La seguridad depende del host. Claude puede pedir aprobación antes de ejecutar tools.

## Construye tu propio MCP server

El patrón es simple:

1. **Identifica datos locales valiosos** (emails, documentos, bases de datos)

2. **Define qué acciones tienen sentido** (buscar, analizar, transformar)

3. **Implementa el servidor MCP**

4. **Conéctalo a tu host favorito** (Claude, Cursor, tu propia app)

MCP es solo un protocolo que conectar la IA con el mundo real.

## Conclusión

Tu asistente de IA ahora puede acceder a TUS datos, usar TUS herramientas, trabajar en TU contexto.

El servidor de WhatsApp es solo una idea ilustrativa. Una vez que entiendes que cualquier programa que hable por STDIN/STDOUT puede ser un servidor MCP las posibilidades son infinitas.

La próxima vez que pienses "ojalá Claude pudiera acceder a...", recuerda: seguramente pueda. Solo necesitas construir el puente.
