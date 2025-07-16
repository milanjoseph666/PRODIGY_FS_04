# PRODIGY_FS_04
Real Time Chat Application 
Project Structure
pgsql
Copy code
RealTimeChatApp/
├── backend/
│   └── server.js
├── frontend/
│   └── index.html
└── README.md

#backend/server.js (Node.js + Socket.io)

const express = require('express');
const http = require('http');
const cors = require('cors');
const socketIo = require('socket.io');

const app = express();
const server = http.createServer(app);

const io = socketIo(server, {
    cors: {
        origin: "*",
        methods: ["GET", "POST"]
    }
});

let users = {};

io.on("connection", (socket) => {
    console.log("User connected:", socket.id);

    socket.on("join", (username) => {
        users[socket.id] = username;
        io.emit("userList", Object.values(users));
        io.emit("notification", `${username} has joined the chat`);
    });

    socket.on("sendMessage", (data) => {
        io.emit("receiveMessage", { message: data.message, sender: users[socket.id] });
    });

    socket.on("sendFile", (fileData) => {
        io.emit("receiveFile", { ...fileData, sender: users[socket.id] });
    });

    socket.on("disconnect", () => {
        io.emit("notification", `${users[socket.id]} left the chat`);
        delete users[socket.id];
        io.emit("userList", Object.values(users));
    });
});

server.listen(5000, () => {
    console.log("Server running on http://localhost:5000");
});

#frontend/index.html (HTML + JS)   

<!DOCTYPE html>
<html>
<head>
    <title>Real-Time Chat App</title>
    <style>
        body { font-family: Arial; background: #f0f0f0; padding: 20px; }
        #chat { margin-top: 20px; display: none; }
        #messages { list-style-type: none; padding: 0; }
        #messages li { margin-bottom: 10px; }
        .notification { color: gray; font-style: italic; }
    </style>
</head>
<body>
    <h2>Real-Time Chat App</h2>
    <input id="username" placeholder="Enter username" />
    <button onclick="join()">Join Chat</button>

    <div id="chat">
        <p><strong>Online Users:</strong> <span id="users"></span></p>
        <ul id="messages"></ul>

        <input id="message" placeholder="Type a message" />
        <button onclick="sendMessage()">Send</button>

        <input type="file" id="fileInput" />
        <button onclick="sendFile()">Send File</button>
    </div>

    <script src="https://cdn.socket.io/4.3.2/socket.io.min.js"></script>
    <script>
        const socket = io("http://localhost:5000");
        let username = "";
        const chat = document.getElementById("chat");

        function join() {
            username = document.getElementById("username").value;
            if (!username) return alert("Enter a username");
            socket.emit("join", username);
            chat.style.display = "block";
        }

        socket.on("receiveMessage", ({ message, sender }) => {
            const li = document.createElement("li");
            li.textContent = `${sender}: ${message}`;
            document.getElementById("messages").appendChild(li);
        });

        socket.on("notification", (note) => {
            const li = document.createElement("li");
            li.textContent = note;
            li.classList.add("notification");
            document.getElementById("messages").appendChild(li);
            notifyUser(note);
        });

        socket.on("userList", (users) => {
            document.getElementById("users").textContent = users.join(", ");
        });

        function sendMessage() {
            const msg = document.getElementById("message").value;
            if (!msg) return;
            socket.emit("sendMessage", { message: msg });
            document.getElementById("message").value = "";
        }

        function sendFile() {
            const file = document.getElementById("fileInput").files[0];
            if (!file) return;

            const reader = new FileReader();
            reader.onload = () => {
                socket.emit("sendFile", {
                    fileName: file.name,
                    fileData: reader.result
                });
            };
            reader.readAsDataURL(file);
        }

        socket.on("receiveFile", ({ fileName, fileData, sender }) => {
            const li = document.createElement("li");
            li.innerHTML = `${sender} sent a file: <a href="${fileData}" download="${fileName}">${fileName}</a>`;
            document.getElementById("messages").appendChild(li);
        });

        // Browser Notification
        function notifyUser(message) {
            if (Notification.permission === "granted") {
                new Notification("Chat Notification", { body: message });
            } else if (Notification.permission !== "denied") {
                Notification.requestPermission().then(permission => {
                    if (permission === "granted") {
                        new Notification("Chat Notification", { body: message });
                    }
                });
            }
        }
    </script>
</body>
</html>

#README.md

# Real-Time Chat Application
## Features
- WebSocket-powered real-time messaging
- User presence tracking
- Browser notifications for join/leave/events
- In-browser chat history
- Basic file sharing

## Optional Features Implemented
- ✅ Chat history (frontend)
- ✅ Notifications (browser API)
- ✅ User presence indicator
- ✅ File sharing (images, docs etc.)

## Run Instructions

### Backend
1. Install Node.js
2. Run the server:
```bash
cd backend
npm install express socket.io cors
node server.js

