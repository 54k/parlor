# parlor

A real-time WebSocket chat server. Rooms, message history, slash-command dispatch. Built on Netty, wired with Guice.

## Why it exists

A study in applying domain-driven design to a network application that is otherwise trivially reactive: connection management, session state, per-room broadcast, and an extensible command vocabulary — each kept in its own layer.

Not a Slack. A minimal multi-room chat server that survives reconnection, persists history, and runs on one JVM.

## Architecture

Three layers:

- **Network** — `NetworkServer` accepts WebSocket upgrades; `NetworkSession` wraps each connection. Codec is standard WebSocket over Netty.
- **Domain** — rooms, users, messages, history. The server is a graph of rooms each holding a set of sessions.
- **Command dispatch** — inbound frames parse to either chat messages or slash commands. Commands register through a Guice module.

Dependency wiring via `google-guice`. Background workers scheduled through `TaskManager` (a thin wrapper over a `ScheduledExecutorService`).

## Build and run

```bash
mvn clean install
java -jar target/parlor.jar
```

Connect a WebSocket client to `ws://localhost:8080/` and send JSON frames:

```json
{"cmd":"join","room":"lobby"}
{"cmd":"say","room":"lobby","text":"hello"}
{"cmd":"history","room":"lobby","limit":50}
```

## Commands

Built-in:

- `/join <room>` — join or create a room.
- `/leave <room>` — leave a room.
- `/say <room> <text>` — send a message.
- `/history <room> [limit]` — replay recent messages.
- `/list` — list rooms.

Add new commands by implementing `Command` and binding it in the Guice module.

## What it doesn't do

- No authentication. Sessions are anonymous; a user id is assigned on connect.
- No federation. Single-node.
- History is in-memory and bounded per-room. Not durable across restarts.
- No encryption at rest. TLS is your reverse proxy's problem.

## License

MIT.
