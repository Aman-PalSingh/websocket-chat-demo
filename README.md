# Original screenshot code — polling and streaming chat

The four JavaScript files (`client.js`, `server.js`, `messaging_api.js`, and `helpers.js`) are transcribed from the screenshots you uploaded. The code has not been silently modified to fix behavior.

## Run

Requires Node.js.

1. Open this folder in VS Code.
2. Install dependencies: `npm install`
3. Terminal 1: `npm start`
4. Terminal 2: `npm run aman`
5. Terminal 3: `npm run rahul`

Type a message and press Enter in either client.

For polling mode, use `npm run aman:poll` and `npm run rahul:poll` instead of the stream commands.

## Important note about the screenshot code

The `app.post('/messages', ...)` handler in the screenshots does not send an HTTP response with `res.send()`, `res.json()`, or `res.end()`. This means an HTTP POST client such as Axios can keep waiting for a response even though the server has stored and broadcast the message. This behavior is preserved because you requested the code as shown in the screenshots.

The screenshot code also uses a plain object (`displayedMessages`) to track IDs already displayed. In the line-input handler, it marks the outgoing ID as displayed before sending the POST request. This is part of the original code.
