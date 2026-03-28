# Angular SSE Demo

A working example of Server-Sent Events (SSE) with Angular and Node.js. The backend pushes real-time health/sensor data (temperature, name, RFID) to the frontend over a persistent HTTP connection — no WebSockets, no polling.

Built this to understand how SSE works compared to WebSockets and long polling. SSE is one-directional (server to client), uses plain HTTP, auto-reconnects on disconnect, and is natively supported by browsers via the `EventSource` API.

## How it works

```
Node.js Server (port 5000)              Angular App (port 4200)
┌─────────────────────────┐              ┌────────────────────────┐
│  GET /events            │◄─────────────│  EventSource connects  │
│                         │              │                        │
│  event: message         │─────5s──────►│  { Name, Temp, RFID }  │
│  event: message         │────10s──────►│  { Name, Temp, RFID }  │
│  event: message         │────15s──────►│  { Name, Temp, RFID }  │
│  event: message         │────20s──────►│  { Name, Temp, RFID }  │
│                         │              │                        │
│  Connection stays open  │──────────────│  UI updates in real-time│
└─────────────────────────┘              └────────────────────────┘
```

- Backend sends 4 events at 5-second intervals with sample health data
- Frontend wraps `EventSource` in an RxJS `Observable` via a custom `SseService`
- Uses Angular's `NgZone` to trigger change detection on incoming events
- Supports reconnection — if the connection drops, `EventSource` auto-reconnects and the server replays missed events using `last-event-id`

## Tech Stack

**Frontend** — Angular 14, RxJS, TypeScript

**Backend** — Node.js (plain `http` module, no Express)

## Running it

```bash
# Start the SSE backend
cd sse-backend
npm install
npm start              # http://localhost:5000

# Start the Angular frontend (separate terminal)
npm install
npm start              # http://localhost:4200
```

Open `http://localhost:4200` — you'll see health data appear every 5 seconds as the server pushes events.

## Key Files

| File | What it does |
|------|-------------|
| `sse-backend/server.js` | Node.js HTTP server, sends SSE events with health data at intervals |
| `src/app/sse.service.ts` | Wraps `EventSource` in an RxJS Observable for Angular consumption |
| `src/app/app.component.ts` | Subscribes to SSE stream, displays received data |

## Sample Event Data

```json
{
  "Name": "Mass Roy",
  "Temparature": "102.5F",
  "RFID": "R546546"
}
```

## Why SSE over WebSockets?

| | SSE | WebSocket |
|---|---|---|
| Direction | Server → Client only | Bidirectional |
| Protocol | Plain HTTP | ws:// (separate protocol) |
| Reconnection | Built-in (automatic) | Manual |
| Browser support | Native `EventSource` API | Native `WebSocket` API |
| Use case | Live feeds, notifications, dashboards | Chat, gaming, real-time collaboration |

SSE is the simpler choice when you only need server-to-client push — no handshake upgrade, works through proxies and firewalls, and the browser handles reconnection for you.
