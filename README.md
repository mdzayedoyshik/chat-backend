# chat-backend

A chat API on Express 5, MongoDB (Mongoose) and Socket.IO. Users log in with email and password and get a JWT. They can start a conversation with another user, send text messages with image or PDF attachments, and join a Socket.IO room per conversation to receive messages live.

It is the server side of the real-time chat I build into Flutter apps, kept small enough to read in one sitting.

## Layout

```
index.js                 Express app + HTTP server + Socket.IO
router/                  /  (login)   /users   /chat
controllers/             request handlers
middlewares/
  auth/                  JWT check for Socket.IO handshakes
  common/                JWT check for HTTP, 404 and error handler
  login/ users/ chat/    express-validator rules per route, multer upload configs
models/                  User, Chat (participants), Message
socket/socket.js         join_conversation / send_message / receive_message
utilities/               single and multi file upload helpers (multer)
```

## HTTP API

| Method | Path | Notes |
|---|---|---|
| POST | `/` | Log in, returns a JWT (valid for 1 day) |
| GET | `/users` | List users |
| POST | `/users` | Create a user, with an optional avatar upload |
| DELETE | `/users/:id` | Delete a user |
| GET | `/chat` | Conversations of the current user |
| POST | `/chat` | Start a conversation |
| GET | `/chat/:id` | Messages in a conversation |
| POST | `/chat/:id` | Send a message. Attachments: JPEG, PNG or PDF, up to 5 MB each |

Every route except login expects `Authorization: Bearer <token>`.

## Socket.IO

The client connects with `{ auth: { token } }`, and the handshake is rejected without a valid JWT.

```js
socket.emit("join_conversation", conversationId);
socket.emit("send_message", { conversationId, message });
socket.on("receive_message", ({ conversationId, message, sender, createdAt }) => { ... });
```

## Running it

```bash
npm install
cp .env.example .env
npm run dev
```

`.env` needs `PORT`, `MONGODB_URL`, `JWT_SECRET`, `COOKIE_SECRET` and, optionally, `CLIENT_ORIGIN` for CORS on the socket server.

## Known gaps

- Messages sent over the socket are broadcast but not saved. Only `POST /chat/:id` persists a message. A real client should call the HTTP endpoint and let the server emit, rather than emitting directly.
- Uploaded files are stored on local disk under `public/uploads`. That does not survive redeploys on most hosts.
- No tests and no rate limiting.
