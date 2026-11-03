# 💬 Node.js Real-Time Chat — Polling & WebSocket Streaming

A lightweight, terminal-based chat application built with **Node.js, Express, HTTP APIs, and WebSockets**. This project demonstrates how multiple clients can exchange messages through a central server using two different communication strategies: **HTTP polling** and **WebSocket streaming**.

The goal is to understand how chat systems work under the hood—from sending an HTTP request to storing messages, fetching updates, and delivering messages to multiple connected clients.

## Features

- **Multiple chat clients** — Run multiple users in separate terminals.
- **HTTP polling** — Fetch new messages from the server at regular intervals.
- **WebSocket streaming** — Receive messages pushed by the server over a persistent connection.
- **REST API** — Send and retrieve messages using HTTP `POST` and `GET` requests.
- **Duplicate message prevention** — Track message IDs to avoid printing the same message multiple times.
- **In-memory message storage** — Keep messages available while the server process is running.
- **Terminal-based interface** — Send and receive messages without a browser or frontend framework.

## Tech Stack

| Technology | Purpose |
|---|---|
| Node.js | JavaScript runtime |
| Express.js | HTTP server and REST endpoints |
| Axios | HTTP requests from clients |
| WebSocket (`ws`) | WebSocket client connection |
| `express-ws` | WebSocket endpoint support in Express |
| `readline` | Read messages from the terminal |

## Architecture

```text
                    Chat Server
                  Express :3001
                        |
             +----------+----------+
             |                     |
       HTTP REST API          WebSocket
       GET /messages        /messages
       POST /messages           |
             |                  |
       +-----+------+-----------+
       |            |           |
    Client A     Client B    Client C
      Aman        Rahul       Another user
       |            |
       +------------+
      Polling or Streaming
```

All clients communicate with the same server. The server stores messages in a shared in-memory array and makes them available to clients.

### Message delivery flow

1. A user types a message and presses Enter.
2. The client creates a message object containing an ID, text, and username.
3. The client sends the message to the server through an HTTP `POST` request.
4. The server stores the message.
5. Other polling clients discover it on their next `GET` request, while streaming clients receive it through WebSocket broadcasts.

## Polling vs. WebSocket Streaming

This project demonstrates two ways to receive messages.

### 1. HTTP Polling

In polling mode, the client requests the message list repeatedly at a fixed interval.

```javascript
function pollMessages() {
  setInterval(getAndDisplayMessages, 3000);
}
```

**How it works:**

- The client fetches messages immediately when it starts.
- It makes another HTTP `GET` request every three seconds.
- The server returns the available messages.
- The client prints only messages it hasn't already displayed.

**Advantages**
- Simple to implement and debug.
- Uses ordinary HTTP requests.
- Can retrieve previously stored messages when reconnecting, provided the server still has them.

**Limitations**
- Updates can be delayed until the next poll.
- Repeated requests create additional HTTP traffic.
- Returning the complete message history on every poll is inefficient as the history grows.

### 2. WebSocket Streaming

In streaming mode, the client establishes a WebSocket connection and keeps it open.

```javascript
const messagingSocket = messagingApi.createMessagingSocket();

messagingSocket.on('message', data => {
  const message = JSON.parse(data);

  const messageAlreadyDisplayed =
    message.id in displayedMessages;

  if (!messageAlreadyDisplayed) {
    displayMessage(message);
  }
});
```

When a new message arrives, the server broadcasts it to connected WebSocket clients.

**Advantages**
- Messages can arrive without waiting for a polling interval.
- No repeated polling requests are required.
- Suitable for interactive applications that need frequent updates.

**Limitations**
- Requires connection management and reconnection handling for production use.
- Disconnected clients don't receive live broadcasts while offline; they need to fetch missed messages after reconnecting.

### Comparison

| Characteristic | HTTP Polling | WebSocket Streaming |
|---|---|---|
| Communication | Repeated HTTP requests | Persistent, bidirectional connection |
| Server sends updates | In response to a request | Over the established connection |
| Update timing | At the next poll | Usually as soon as the server broadcasts |
| Repeated requests | Yes | No repeated polling required |
| Connection stays open | Not as a persistent application connection | Yes |
| Best suited for | Simple periodic updates | Interactive, low-latency updates |

## 📂 Project Structure

```text
.
├── client.js          # Terminal client and message handling
├── server.js          # Express server, REST API and WebSocket endpoint
├── messaging_api.js   # HTTP and WebSocket client helpers
├── helpers.js         # Utility functions, including message ID generation
├── package.json       # Dependencies and npm scripts
├── package-lock.json  # Dependency lockfile (if generated)
└── README.md          # Project documentation
```

## ⚙️ Prerequisites

- [Node.js](https://nodejs.org/) — a current LTS version is recommended.
- npm — included with Node.js.
- A terminal capable of running multiple shell sessions.

## 🚀 Getting Started

### 1. Clone the repository

Replace the placeholder with your GitHub repository URL.

```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPOSITORY.git
cd YOUR_REPOSITORY
```

### 2. Install dependencies

```bash
npm install
```

### 3. Start the server

Open your first terminal and run:

```bash
node server.js
```

Expected output:

```text
Listening on port 3001!
```

Keep this terminal running. It hosts the HTTP API and WebSocket endpoint.

### 4. Start the first chat client

Open a second terminal in the project directory.

**Polling mode:**

```bash
NAME=Aman MODE=poll node client.js
```

**Streaming mode:**

```bash
NAME=Aman MODE=stream node client.js
```

### 5. Start another user

Open a third terminal and run:

```bash
NAME=Rahul MODE=poll node client.js
```

Or use streaming mode:

```bash
NAME=Rahul MODE=stream node client.js
```

Now type a message in one terminal and press Enter. The other clients can receive it through their selected delivery mechanism.

> **Tip:** You can mix modes. For example, run Aman in streaming mode and Rahul in polling mode to compare how each client receives the same message.

## 🧪 Experiment With the Different Modes

### Polling mode

```bash
NAME=Rahul MODE=poll node client.js
```

The client fetches messages immediately and then polls every three seconds.

### Streaming mode

```bash
NAME=Aman MODE=stream node client.js
```

The client fetches the existing messages once and then listens for new messages over WebSocket.

### No `MODE` variable

```bash
NAME=Rahul node client.js
```

In the current implementation, neither the polling nor streaming initialization branch runs. The terminal input handler is still registered, so the client can attempt to send messages, but it won't automatically retrieve incoming messages.

### Stop a client or server

Press `Ctrl+C` in its terminal.

## 🔌 API Endpoints

The server listens on port `3001`.

| Method | Endpoint | Purpose |
|---|---|---|
| `GET` | `/messages` | Retrieve stored messages |
| `POST` | `/messages` | Store a new message |
| WebSocket | `/messages` | Connect to receive live message broadcasts |

### Example message

```json
{
  "id": 1234,
  "text": "Hello, everyone!",
  "username": "Aman"
}
```

### How message IDs prevent duplicates

Each client maintains its own `displayedMessages` object.

When a client sends a message, it marks that message's ID as already displayed. When it receives a message from polling or WebSocket streaming, it checks the ID before printing it.

This helps prevent:
- A sender from printing its own message a second time.
- A client from printing the same stored message during every polling cycle.
- A