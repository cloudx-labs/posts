# WhatsApp + MCP: automatic audio transcription

MCP (Model Context Protocol) can look complicated until you ship something real with it. So let's take it for a spin with something useful: expose your WhatsApp voice notes with your own MCP server and turn them into transcripts.

## What is MCP?

MCP is a connection standard that links AI agents with external systems.

It has a server and a client, and they have two different ways to talk to each other:

- stdio (stdin/stdout): the standard Unix mechanism for a process to receive or hand off data to the environment or another process.
- Server-Sent Events (SSE): an HTTP mechanism where the server keeps the connection open and streams events to the client (one-way).

![STDIN/STDOUT vs SSE in MCP](./assets/mcp_whatsapp_2.png)

## MCP architecture

- Host: Claude Desktop / Cursor / any AI agent. It coordinates the LLM, spins up MCP clients, and shows results.
- MCP Client: an implementation embedded in the host that connects to your server. It speaks the protocol, opens/manages the connection, and sends/receives requests.
- MCP Server: your program that exposes tools/resources. It runs actions and returns data/events to the client.

The server exposes three things:

- **Tools**: actions it can execute (transcribe audio, search files)
- **Resources**: data it can provide (contact list, recent files)
- **Prompts**: interaction templates (we skip them in this example)

![MCP architecture: Host -> MCP Client -> MCP Server](./assets/mcp_whatsapp_1.png)

## Building the WhatsApp MCP

WhatsApp Desktop on macOS stores everything locally: an SQLite database with chats and folders containing the media files.

Our MCP server will:

1. Read the WhatsApp database
2. Find audio files per contact
3. Transcribe them with OpenAI Whisper (SDK)
4. Send the text back to Claude

The working code lives in the repository: [pending link](REPO_URL_PENDING). Let's walk through the key pieces.

### The STDIN/STDOUT connection

```typescript
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const transport = new StdioServerTransport();
await this.server.connect(transport);
```

With that the server listens to every client request on STDIN and replies through STDOUT.

We pick stdio because this MCP server runs locally. It's the simplest and most stable transport on desktop/CLI: no open ports, no HTTP dependency, avoids CORS/firewalls, and hosts (Claude Desktop/Cursor) support it natively. SSE makes sense when the server lives remotely behind HTTP.

### Exposing capabilities

```typescript
this.server = new Server(
  {
    name: "whatsapp-audio-mcp",
    version: "1.0.0",
  },
  {
    capabilities: {
      resources: {}, // We will expose data
      tools: {}, // We will expose actions
    },
  }
);
```

### Resources: the data we expose

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

Each resource ships with a unique URI. When Claude asks for `whatsapp://contacts`, our server knows what to return.

### Tools: the actions available

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

The schema follows JSON Schema. With it, Claude knows which parameters to send.

## The engine: accessing WhatsApp

WhatsApp Desktop keeps everything under predictable paths:

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

The database is SQLite:

```typescript
const query = `
  SELECT DISTINCT 
    ZCONTACTJID as jid,
    ZPARTNERNAME as name,
    ZLASTMESSAGEDATE as lastMessageDate
  FROM ZWACHATSESSION
  WHERE ZPARTNERNAME IS NOT NULL
  AND ZCONTACTJID NOT LIKE '%@g.us'  -- Exclude groups
`;
```

Audio files are organized per contact. We scan recursively:

```typescript
const audioExtensions = [".opus", ".m4a", ".mp3", ".aac", ".wav"];

async function scanDirectory(dir: string): Promise<void> {
  const entries = await fs.readdir(dir, { withFileTypes: true });

  for (const entry of entries) {
    if (audioExtensions.some((ext) => entry.name.endsWith(ext))) {
      // Found an audio file
      audioFiles.push({
        path: fullPath,
        filename: entry.name,
        modifiedDate: stats.mtime.toISOString(),
      });
    }
  }
}
```

## The transcription: FFmpeg + OpenAI Whisper (SDK)

WhatsApp ships audio in Opus, but OpenAI Whisper prefers MP3. We lean on FFmpeg:

```typescript
const ffmpeg = spawn("ffmpeg", [
  "-i",
  inputPath, // WhatsApp Opus audio
  "-acodec",
  "mp3",
  "-b:a",
  "128k",
  outputPath, // Temporary MP3
]);
```

Then we transcribe with OpenAI Whisper (SDK):

```typescript
import OpenAI from "openai";
import fs from "node:fs";

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

const transcription = await openai.audio.transcriptions.create({
  file: fs.createReadStream(outputPath), // Temporary MP3
  model: "whisper-1",
});

const transcriptionText = transcription.text;
```

## Configuring Claude (the client)

In the Claude Desktop config we add:

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

Claude can now invoke our server whenever it needs to.

## MCP in action

The user asks Claude:

> "What did Lucas say in the last voice note he sent me?"

Claude automatically:

1. Calls `getRecentAudio(contactName: "lucas")`
2. Receives the audio file path
3. Calls `transcribeAudio(audioPath: "/path/to/audio.opus")`
4. Receives the transcription
5. Summarizes or shows the full text

The transcription flows through the OpenAI API; the temporary MP3 is sent to get the text back. Claude orchestrates; your server prepares the file and makes the call.

![Claude + MCP WhatsApp: transcription example](./assets/mcp_whatsapp_3.png)

## Limitations: macOS only

This server is macOS only. The WhatsApp paths are specific to Mac.

It depends on:

- WhatsApp Desktop installed
- FFmpeg (`brew install ffmpeg`)
- OpenAI SDK (`npm i openai`) with `OPENAI_API_KEY` configured
- Internet connection

We also skip Prompts and Resource Templates.

Security depends on the host. Claude can ask for approval before it runs tools.

## Build your own MCP server

The pattern is simple:

1. **Identify valuable local data** (email, documents, databases)
2. **Define which actions make sense** (search, analyze, transform)
3. **Implement the MCP server**
4. **Hook it into your favorite host** (Claude, Cursor, your own app)

MCP is just a protocol that connects AI with the real world.

## Conclusion

Your AI assistant can now reach YOUR data, use YOUR tools, work in YOUR context.

The WhatsApp server is just one illustrative idea. Once you realize any program that speaks STDIN/STDOUT can be an MCP server, the possibilities get wild.

Next time you think "I wish Claude could access...", remember: it probably can. You just need to build the bridge.
