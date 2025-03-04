# Setting Up a WebSocket Server Using `socket.io` (Singleton Approach)

## Explanation of io and socket in Easy Language

### 1️⃣ What is io?
`io` is the WebSocket server itself. It manages all client connections and allows broadcasting messages to all connected users.

Think of `io` as a chat room manager who handles all users joining, leaving, and sending messages.
When a new client connects, `io` detects it and gives them a unique socket (ID).

#### Example:
```ts
this.io.on("connection", (socket) => { 
  // Handle new connection
});
```

This means:
👉 "When someone joins (connects), give them a socket to communicate."

---

### 2️⃣ What is socket?
`socket` represents a single connected user. It allows us to send/receive messages between that specific user and the server.

Every time a user connects, `io` assigns them a socket.
This socket is unique to that user, like a personal walkie-talkie.

#### Example:
```ts
socket.on("message", (data) => {
  console.log("Received message:", data);
  socket.broadcast.emit("message", data);
});
```

This means:
👉 "When this specific user sends a message, share it with all other users except them."

---

### 📌 Summary
- **`io` (WebSocket Server):** Manages all clients.
- **`socket` (Single Connection):** Represents one specific user.
`

### **Steps:**
1. Install dependencies: `npm install express socket.io`
2. Create a singleton `WebSocketServer` class.
3. Ensure the same instance is used across the app.
4. Handle client connections and events.

---

### **1️⃣ Install Dependencies**
```sh
npm install express socket.io
```

---

### **2️⃣ WebSocket Singleton Implementation**
Create a new file **`WebSocketServer.ts`**:
```ts
import { Server as HttpServer } from "http";
import { Server as SocketIOServer, Socket } from "socket.io";

class WebSocketServer {
  private static instance: WebSocketServer;
  private io: SocketIOServer;

  private constructor(server: HttpServer) {
    this.io = new SocketIOServer(server, {
      cors: {
        origin: "*", // Adjust based on your needs
      },
    });

    this.setupListeners();
  }

  public static initialize(server: HttpServer): WebSocketServer {
    if (!WebSocketServer.instance) {
      WebSocketServer.instance = new WebSocketServer(server);
    }
    return WebSocketServer.instance;
  }

  public static getInstance(): WebSocketServer {
    if (!WebSocketServer.instance) {
      throw new Error("WebSocketServer not initialized. Call initialize(server) first.");
    }
    return WebSocketServer.instance;
  }

  private setupListeners() {
    this.io.on("connection", (socket: Socket) => {
      console.log(`Client connected: ${socket.id}`);

      socket.on("message", (data) => {
        console.log("Received message:", data);
        socket.broadcast.emit("message", data); // Broadcast to all clients, except self
      });

      socket.on("disconnect", () => {
        console.log(`Client disconnected: ${socket.id}`);
      });
    });
  }

  public getIO(): SocketIOServer {
    return this.io;
  }
}

export default WebSocketServer;
```

---

### **3️⃣ Setup Express and HTTP Server**
Modify your **`server.ts`** or **`index.ts`**:
```ts
import express from "express";
import http from "http";
import WebSocketServer from "./WebSocketServer"; // Import the singleton class

const app = express();
const server = http.createServer(app);

// Initialize WebSocket server
WebSocketServer.initialize(server);

app.get("/", (req, res) => {
  res.send("WebSocket Server is Running...");
});

const PORT = 5000;
server.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

---

### **4️⃣ Connecting from Client**
If you're using a **frontend (React, Vanilla JS, etc.)**, connect using:

```js
import { io } from "socket.io-client";

const socket = io("http://localhost:5000");

socket.on("connect", () => {
  console.log("Connected to WebSocket Server");
});

socket.emit("message", "Hello from client!");

socket.on("message", (data) => {
  console.log("Received:", data);
});
```

---

### **Why Singleton?**
1. **Ensures only one WebSocket server instance** across the app.
2. **Prevents multiple WebSocket instances** from being created by mistake.
3. **Allows easy access via `getInstance()`** from other parts of the app.

This approach keeps WebSocket handling **modular**, **scalable**, and **maintainable**. 🚀

# Setup client-side.

## `SocketProvider.tsx`

```ts
"use client";
import React, { useCallback, useContext, useEffect, useState } from "react";
import { io, Socket } from "socket.io-client";

interface SocketProviderProps {
  children?: React.ReactNode;
}

interface ISocketContext {
  sendMessage: (msg: string) => void;
  messages: string[];
  socketId: string | null;
}

const SocketContext = React.createContext<ISocketContext | null>(null);

export const useSocket = () => {
  const state = useContext(SocketContext);
  if (!state) throw new Error("SocketContext is undefined");

  return state;
};

export const SocketProvider: React.FC<SocketProviderProps> = ({ children }) => {
  const [socket, setSocket] = useState<Socket | null>(null);
  const [messages, setMessages] = useState<string[]>([]);
  const [socketId, setSocketId] = useState<string | null>(null);

  // Function to send message
  const sendMessage: ISocketContext["sendMessage"] = useCallback(
    (msg) => {
      console.log("Sending message:", msg);
      if (socket) {
        socket.emit("message", msg);
      }
    },
    [socket]
  );

  // Function to handle incoming messages
  const onMessageReceived = useCallback((msg: string) => {
    console.log("Received from server:", msg);
    setMessages((prev) => [...prev, msg]);
  }, []);

  useEffect(() => {
    // Connect to the WebSocket server
    const _socket = io("http://localhost:8000");

    _socket.on("connect", () => {
      console.log("Connected with socket ID:", _socket.id);
      setSocketId(_socket.id);
    });

    _socket.on("message", onMessageReceived);

    setSocket(_socket);

    return () => {
      _socket.off("message", onMessageReceived);
      _socket.disconnect();
      setSocket(null);
      setSocketId(null);
    };
  }, []);

  return (
    <SocketContext.Provider value={{ sendMessage, messages, socketId }}>
      {children}
    </SocketContext.Provider>
  );
};
```
## How to use in any component

```ts
import { useSocket } from "./SocketProvider";

const ChatComponent = () => {
  const { sendMessage, messages, socketId } = useSocket();

  return (
    <div>
      <h2>Socket ID: {socketId}</h2>
      <button onClick={() => sendMessage("Hello Server!")}>Send</button>
      <ul>
        {messages.map((msg, idx) => (
          <li key={idx}>{msg}</li>
        ))}
      </ul>
    </div>
  );
};

export default ChatComponent;
```

# FLOW WHEN WE USE PUB-SUB ARCHITECTURE
  ![image](https://github.com/user-attachments/assets/34e55437-d36e-4c2d-9005-d0999eaa254d)

