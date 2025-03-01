# Setting Up a WebSocket Server Using `socket.io` (Singleton Approach)

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
        socket.broadcast.emit("message", data); // Broadcast to all clients
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
