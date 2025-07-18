
# 🌐 Real-time Communication with WebSockets

## Table of Contents
- [Introduction to WebSockets](#introduction-to-websockets)
- [WebSocket vs HTTP](#websocket-vs-http)
- [Basic WebSocket Implementation](#basic-websocket-implementation)
- [Socket.io Framework](#socketio-framework)
- [Real-time Chat Application](#real-time-chat-application)
- [Broadcasting and Rooms](#broadcasting-and-rooms)
- [Error Handling and Reconnection](#error-handling-and-reconnection)
- [Security Considerations](#security-considerations)

---

## 🚀 Introduction to WebSockets

WebSockets provide full-duplex communication channels over a single TCP connection, enabling real-time data exchange between client and server.

### Key Features:
- **Real-time**: Instant bidirectional communication
- **Low Latency**: Minimal overhead after connection
- **Persistent**: Maintains connection until closed
- **Cross-Platform**: Works across different devices and browsers

### Common Use Cases:
- Live chat applications
- Real-time gaming
- Live streaming and notifications
- Collaborative editing
- Financial trading platforms
- IoT device monitoring

---

## ⚖️ WebSocket vs HTTP

| Feature | HTTP | WebSocket |
|---------|------|-----------|
| **Communication** | Request-Response | Full-duplex |
| **Connection** | Stateless | Persistent |
| **Overhead** | Headers on every request | Minimal after handshake |
| **Real-time** | Polling required | Native support |
| **Caching** | Built-in | Not applicable |

### HTTP Polling vs WebSockets
```javascript
// HTTP Polling (inefficient)
setInterval(async () => {
    const response = await fetch('/api/messages');
    const messages = await response.json();
    updateUI(messages);
}, 1000); // Poll every second

// WebSocket (efficient)
const socket = new WebSocket('ws://localhost:3000');
socket.onmessage = (event) => {
    const message = JSON.parse(event.data);
    updateUI(message);
};
```

---

## 🔧 Basic WebSocket Implementation

### Client-Side WebSocket
```javascript
class WebSocketClient {
    constructor(url) {
        this.url = url;
        this.socket = null;
        this.reconnectAttempts = 0;
        this.maxReconnectAttempts = 5;
        this.reconnectInterval = 1000;
        
        this.connect();
    }
    
    connect() {
        try {
            this.socket = new WebSocket(this.url);
            this.setupEventListeners();
        } catch (error) {
            console.error('WebSocket connection failed:', error);
            this.handleReconnect();
        }
    }
    
    setupEventListeners() {
        this.socket.onopen = (event) => {
            console.log('WebSocket connected:', event);
            this.reconnectAttempts = 0;
            this.onConnect();
        };
        
        this.socket.onmessage = (event) => {
            try {
                const data = JSON.parse(event.data);
                this.onMessage(data);
            } catch (error) {
                console.error('Error parsing message:', error);
            }
        };
        
        this.socket.onclose = (event) => {
            console.log('WebSocket closed:', event.code, event.reason);
            this.onClose(event);
            
            if (!event.wasClean) {
                this.handleReconnect();
            }
        };
        
        this.socket.onerror = (error) => {
            console.error('WebSocket error:', error);
            this.onError(error);
        };
    }
    
    send(data) {
        if (this.socket.readyState === WebSocket.OPEN) {
            this.socket.send(JSON.stringify(data));
        } else {
            console.error('WebSocket is not open');
        }
    }
    
    close() {
        if (this.socket) {
            this.socket.close(1000, 'Client closing connection');
        }
    }
    
    handleReconnect() {
        if (this.reconnectAttempts < this.maxReconnectAttempts) {
            this.reconnectAttempts++;
            console.log(`Reconnecting... (${this.reconnectAttempts}/${this.maxReconnectAttempts})`);
            
            setTimeout(() => {
                this.connect();
            }, this.reconnectInterval * this.reconnectAttempts);
        } else {
            console.error('Max reconnection attempts reached');
        }
    }
    
    // Override these methods in your implementation
    onConnect() { console.log('Connected to WebSocket server'); }
    onMessage(data) { console.log('Received message:', data); }
    onClose(event) { console.log('Connection closed'); }
    onError(error) { console.error('WebSocket error:', error); }
}

// Usage
const client = new WebSocketClient('ws://localhost:3000');

client.onMessage = (data) => {
    console.log('New message:', data);
    // Handle incoming messages
};

// Send a message
client.send({
    type: 'chat',
    message: 'Hello, World!',
    timestamp: Date.now()
});
```

### Server-Side WebSocket (Node.js)
```javascript
const WebSocket = require('ws');
const http = require('http');

class WebSocketServer {
    constructor(port = 3000) {
        this.port = port;
        this.clients = new Map();
        this.rooms = new Map();
        
        this.setupServer();
    }
    
    setupServer() {
        const server = http.createServer();
        this.wss = new WebSocket.Server({ server });
        
        this.wss.on('connection', (ws, req) => {
            const clientId = this.generateClientId();
            const clientInfo = {
                id: clientId,
                ws: ws,
                rooms: new Set(),
                lastPing: Date.now()
            };
            
            this.clients.set(clientId, clientInfo);
            console.log(`Client ${clientId} connected`);
            
            // Send welcome message
            this.sendToClient(clientId, {
                type: 'welcome',
                clientId: clientId,
                message: 'Connected to WebSocket server'
            });
            
            ws.on('message', (message) => {
                this.handleMessage(clientId, message);
            });
            
            ws.on('close', (code, reason) => {
                console.log(`Client ${clientId} disconnected: ${code} ${reason}`);
                this.handleDisconnect(clientId);
            });
            
            ws.on('error', (error) => {
                console.error(`Client ${clientId} error:`, error);
            });
            
            // Setup ping/pong for connection health
            ws.on('pong', () => {
                const client = this.clients.get(clientId);
                if (client) {
                    client.lastPing = Date.now();
                }
            });
        });
        
        // Start ping interval
        this.startPingInterval();
        
        server.listen(this.port, () => {
            console.log(`WebSocket server running on port ${this.port}`);
        });
    }
    
    generateClientId() {
        return 'client_' + Math.random().toString(36).substr(2, 9);
    }
    
    handleMessage(clientId, message) {
        try {
            const data = JSON.parse(message);
            console.log(`Message from ${clientId}:`, data);
            
            switch (data.type) {
                case 'join_room':
                    this.joinRoom(clientId, data.room);
                    break;
                case 'leave_room':
                    this.leaveRoom(clientId, data.room);
                    break;
                case 'chat':
                    this.handleChatMessage(clientId, data);
                    break;
                case 'broadcast':
                    this.broadcast(data, clientId);
                    break;
                default:
                    console.log(`Unknown message type: ${data.type}`);
            }
        } catch (error) {
            console.error('Error parsing message:', error);
        }
    }
    
    sendToClient(clientId, data) {
        const client = this.clients.get(clientId);
        if (client && client.ws.readyState === WebSocket.OPEN) {
            client.ws.send(JSON.stringify(data));
        }
    }
    
    broadcast(data, excludeClientId = null) {
        this.clients.forEach((client, clientId) => {
            if (clientId !== excludeClientId && client.ws.readyState === WebSocket.OPEN) {
                client.ws.send(JSON.stringify(data));
            }
        });
    }
    
    joinRoom(clientId, roomName) {
        const client = this.clients.get(clientId);
        if (!client) return;
        
        client.rooms.add(roomName);
        
        if (!this.rooms.has(roomName)) {
            this.rooms.set(roomName, new Set());
        }
        this.rooms.get(roomName).add(clientId);
        
        console.log(`Client ${clientId} joined room ${roomName}`);
        
        // Notify room members
        this.broadcastToRoom(roomName, {
            type: 'user_joined',
            clientId: clientId,
            room: roomName,
            message: `User ${clientId} joined the room`
        }, clientId);
        
        // Send confirmation to client
        this.sendToClient(clientId, {
            type: 'room_joined',
            room: roomName,
            members: Array.from(this.rooms.get(roomName))
        });
    }
    
    leaveRoom(clientId, roomName) {
        const client = this.clients.get(clientId);
        if (!client) return;
        
        client.rooms.delete(roomName);
        
        if (this.rooms.has(roomName)) {
            this.rooms.get(roomName).delete(clientId);
            
            if (this.rooms.get(roomName).size === 0) {
                this.rooms.delete(roomName);
            } else {
                this.broadcastToRoom(roomName, {
                    type: 'user_left',
                    clientId: clientId,
                    room: roomName,
                    message: `User ${clientId} left the room`
                });
            }
        }
        
        console.log(`Client ${clientId} left room ${roomName}`);
    }
    
    broadcastToRoom(roomName, data, excludeClientId = null) {
        if (!this.rooms.has(roomName)) return;
        
        this.rooms.get(roomName).forEach(clientId => {
            if (clientId !== excludeClientId) {
                this.sendToClient(clientId, data);
            }
        });
    }
    
    handleChatMessage(clientId, data) {
        const message = {
            type: 'chat_message',
            clientId: clientId,
            message: data.message,
            timestamp: Date.now(),
            room: data.room
        };
        
        if (data.room) {
            this.broadcastToRoom(data.room, message);
        } else {
            this.broadcast(message, clientId);
        }
    }
    
    handleDisconnect(clientId) {
        const client = this.clients.get(clientId);
        if (!client) return;
        
        // Remove from all rooms
        client.rooms.forEach(roomName => {
            this.leaveRoom(clientId, roomName);
        });
        
        this.clients.delete(clientId);
    }
    
    startPingInterval() {
        setInterval(() => {
            const now = Date.now();
            this.clients.forEach((client, clientId) => {
                if (now - client.lastPing > 30000) {
                    console.log(`Client ${clientId} ping timeout`);
                    client.ws.terminate();
                } else if (client.ws.readyState === WebSocket.OPEN) {
                    client.ws.ping();
                }
            });
        }, 10000);
    }
}

// Start the server
const server = new WebSocketServer(3000);
```

---

## 🚀 Socket.io Framework

Socket.io is a popular library that provides real-time bidirectional event-based communication.

### Server Setup
```javascript
const express = require('express');
const http = require('http');
const socketIo = require('socket.io');
const path = require('path');

const app = express();
const server = http.createServer(app);
const io = socketIo(server, {
    cors: {
        origin: "*",
        methods: ["GET", "POST"]
    }
});

// Serve static files
app.use(express.static(path.join(__dirname, 'public')));

// Socket.io connection handling
io.on('connection', (socket) => {
    console.log('User connected:', socket.id);
    
    // Join room
    socket.on('join_room', (room) => {
        socket.join(room);
        socket.emit('joined_room', room);
        socket.to(room).emit('user_joined', {
            userId: socket.id,
            message: `User ${socket.id} joined the room`
        });
        console.log(`User ${socket.id} joined room ${room}`);
    });
    
    // Leave room
    socket.on('leave_room', (room) => {
        socket.leave(room);
        socket.to(room).emit('user_left', {
            userId: socket.id,
            message: `User ${socket.id} left the room`
        });
        console.log(`User ${socket.id} left room ${room}`);
    });
    
    // Handle chat messages
    socket.on('chat_message', (data) => {
        const message = {
            id: Date.now(),
            userId: socket.id,
            username: data.username || 'Anonymous',
            message: data.message,
            timestamp: new Date().toISOString(),
            room: data.room
        };
        
        if (data.room) {
            // Send to room
            io.to(data.room).emit('new_message', message);
        } else {
            // Broadcast to all
            io.emit('new_message', message);
        }
    });
    
    // Handle typing indicators
    socket.on('typing_start', (data) => {
        socket.to(data.room).emit('user_typing', {
            userId: socket.id,
            username: data.username
        });
    });
    
    socket.on('typing_stop', (data) => {
        socket.to(data.room).emit('user_stop_typing', {
            userId: socket.id
        });
    });
    
    // Handle disconnect
    socket.on('disconnect', () => {
        console.log('User disconnected:', socket.id);
    });
    
    // Custom events
    socket.on('game_move', (data) => {
        socket.to(data.room).emit('opponent_move', data);
    });
    
    socket.on('file_share', (data) => {
        socket.to(data.room).emit('file_received', {
            filename: data.filename,
            size: data.size,
            type: data.type,
            userId: socket.id
        });
    });
});

server.listen(3000, () => {
    console.log('Server running on port 3000');
});
```

### Client-Side Socket.io
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Real-time Chat</title>
    <script src="/socket.io/socket.io.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .chat-container {
            border: 1px solid #ddd;
            height: 400px;
            overflow-y: auto;
            padding: 10px;
            margin-bottom: 20px;
            background: #f9f9f9;
        }
        
        .message {
            margin-bottom: 10px;
            padding: 8px;
            background: white;
            border-radius: 5px;
            border-left: 3px solid #007bff;
        }
        
        .message.own {
            background: #e3f2fd;
            border-left-color: #4CAF50;
            margin-left: 20px;
        }
        
        .message-header {
            font-size: 12px;
            color: #666;
            margin-bottom: 4px;
        }
        
        .typing-indicator {
            font-style: italic;
            color: #666;
            font-size: 14px;
        }
        
        .input-container {
            display: flex;
            gap: 10px;
        }
        
        .message-input {
            flex: 1;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 5px;
        }
        
        .send-btn {
            padding: 10px 20px;
            background: #007bff;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
        
        .room-controls {
            margin-bottom: 20px;
        }
        
        .room-input {
            padding: 8px;
            margin-right: 10px;
            border: 1px solid #ddd;
            border-radius: 3px;
        }
        
        .room-btn {
            padding: 8px 15px;
            background: #28a745;
            color: white;
            border: none;
            border-radius: 3px;
            cursor: pointer;
            margin-right: 5px;
        }
    </style>
</head>
<body>
    <h1>Real-time Chat with Socket.io</h1>
    
    <div class="room-controls">
        <input type="text" id="usernameInput" class="room-input" placeholder="Enter username" value="User123">
        <input type="text" id="roomInput" class="room-input" placeholder="Enter room name" value="general">
        <button id="joinBtn" class="room-btn">Join Room</button>
        <button id="leaveBtn" class="room-btn" style="background: #dc3545;">Leave Room</button>
    </div>
    
    <div id="chatContainer" class="chat-container"></div>
    <div id="typingIndicator" class="typing-indicator"></div>
    
    <div class="input-container">
        <input type="text" id="messageInput" class="message-input" placeholder="Type your message...">
        <button id="sendBtn" class="send-btn">Send</button>
    </div>
    
    <script>
        class ChatApp {
            constructor() {
                this.socket = io();
                this.currentRoom = null;
                this.username = 'User123';
                this.typingTimeout = null;
                
                this.bindEvents();
                this.setupSocketListeners();
            }
            
            bindEvents() {
                const joinBtn = document.getElementById('joinBtn');
                const leaveBtn = document.getElementById('leaveBtn');
                const sendBtn = document.getElementById('sendBtn');
                const messageInput = document.getElementById('messageInput');
                const usernameInput = document.getElementById('usernameInput');
                
                joinBtn.addEventListener('click', () => this.joinRoom());
                leaveBtn.addEventListener('click', () => this.leaveRoom());
                sendBtn.addEventListener('click', () => this.sendMessage());
                
                messageInput.addEventListener('keypress', (e) => {
                    if (e.key === 'Enter') {
                        this.sendMessage();
                    } else {
                        this.handleTyping();
                    }
                });
                
                messageInput.addEventListener('input', () => this.handleTyping());
                messageInput.addEventListener('blur', () => this.stopTyping());
                
                usernameInput.addEventListener('change', (e) => {
                    this.username = e.target.value || 'Anonymous';
                });
            }
            
            setupSocketListeners() {
                this.socket.on('connect', () => {
                    console.log('Connected to server');
                    this.addSystemMessage('Connected to server');
                });
                
                this.socket.on('disconnect', () => {
                    console.log('Disconnected from server');
                    this.addSystemMessage('Disconnected from server');
                });
                
                this.socket.on('joined_room', (room) => {
                    this.currentRoom = room;
                    this.addSystemMessage(`Joined room: ${room}`);
                });
                
                this.socket.on('user_joined', (data) => {
                    this.addSystemMessage(data.message);
                });
                
                this.socket.on('user_left', (data) => {
                    this.addSystemMessage(data.message);
                });
                
                this.socket.on('new_message', (message) => {
                    this.addMessage(message);
                });
                
                this.socket.on('user_typing', (data) => {
                    this.showTypingIndicator(data.username);
                });
                
                this.socket.on('user_stop_typing', () => {
                    this.hideTypingIndicator();
                });
            }
            
            joinRoom() {
                const roomInput = document.getElementById('roomInput');
                const room = roomInput.value.trim();
                
                if (room) {
                    if (this.currentRoom) {
                        this.socket.emit('leave_room', this.currentRoom);
                    }
                    this.socket.emit('join_room', room);
                    this.clearChat();
                }
            }
            
            leaveRoom() {
                if (this.currentRoom) {
                    this.socket.emit('leave_room', this.currentRoom);
                    this.currentRoom = null;
                    this.addSystemMessage('Left the room');
                }
            }
            
            sendMessage() {
                const messageInput = document.getElementById('messageInput');
                const message = messageInput.value.trim();
                
                if (message) {
                    this.socket.emit('chat_message', {
                        message: message,
                        username: this.username,
                        room: this.currentRoom
                    });
                    
                    messageInput.value = '';
                    this.stopTyping();
                }
            }
            
            handleTyping() {
                if (this.currentRoom) {
                    this.socket.emit('typing_start', {
                        username: this.username,
                        room: this.currentRoom
                    });
                    
                    clearTimeout(this.typingTimeout);
                    this.typingTimeout = setTimeout(() => {
                        this.stopTyping();
                    }, 1000);
                }
            }
            
            stopTyping() {
                if (this.currentRoom) {
                    this.socket.emit('typing_stop', {
                        room: this.currentRoom
                    });
                }
                clearTimeout(this.typingTimeout);
            }
            
            addMessage(message) {
                const chatContainer = document.getElementById('chatContainer');
                const messageDiv = document.createElement('div');
                const isOwnMessage = message.userId === this.socket.id;
                
                messageDiv.className = `message ${isOwnMessage ? 'own' : ''}`;
                messageDiv.innerHTML = `
                    <div class="message-header">
                        ${message.username} - ${new Date(message.timestamp).toLocaleTimeString()}
                    </div>
                    <div>${this.escapeHtml(message.message)}</div>
                `;
                
                chatContainer.appendChild(messageDiv);
                chatContainer.scrollTop = chatContainer.scrollHeight;
            }
            
            addSystemMessage(text) {
                const chatContainer = document.getElementById('chatContainer');
                const messageDiv = document.createElement('div');
                
                messageDiv.style.cssText = 'color: #666; font-style: italic; margin: 10px 0; text-align: center;';
                messageDiv.textContent = text;
                
                chatContainer.appendChild(messageDiv);
                chatContainer.scrollTop = chatContainer.scrollHeight;
            }
            
            showTypingIndicator(username) {
                const indicator = document.getElementById('typingIndicator');
                indicator.textContent = `${username} is typing...`;
            }
            
            hideTypingIndicator() {
                const indicator = document.getElementById('typingIndicator');
                indicator.textContent = '';
            }
            
            clearChat() {
                const chatContainer = document.getElementById('chatContainer');
                chatContainer.innerHTML = '';
            }
            
            escapeHtml(text) {
                const div = document.createElement('div');
                div.textContent = text;
                return div.innerHTML;
            }
        }
        
        // Initialize the chat app
        const chatApp = new ChatApp();
    </script>
</body>
</html>
```

---

## 🎮 Real-time Game Example

```javascript
// Game Server (Socket.io)
class GameServer {
    constructor(io) {
        this.io = io;
        this.games = new Map();
        this.waitingPlayers = [];
        
        this.setupSocketHandlers();
    }
    
    setupSocketHandlers() {
        this.io.on('connection', (socket) => {
            console.log('Player connected:', socket.id);
            
            socket.on('find_game', (playerData) => {
                this.findGame(socket, playerData);
            });
            
            socket.on('game_move', (data) => {
                this.handleMove(socket, data);
            });
            
            socket.on('disconnect', () => {
                this.handleDisconnect(socket);
            });
        });
    }
    
    findGame(socket, playerData) {
        socket.playerData = playerData;
        
        if (this.waitingPlayers.length > 0) {
            // Match with waiting player
            const opponent = this.waitingPlayers.pop();
            this.createGame(socket, opponent);
        } else {
            // Add to waiting list
            this.waitingPlayers.push(socket);
            socket.emit('waiting_for_opponent');
        }
    }
    
    createGame(player1, player2) {
        const gameId = `game_${Date.now()}`;
        const game = {
            id: gameId,
            players: [player1, player2],
            board: Array(9).fill(null), // Tic-tac-toe board
            currentPlayer: 0,
            status: 'playing'
        };
        
        this.games.set(gameId, game);
        
        // Assign game to players
        player1.gameId = gameId;
        player2.gameId = gameId;
        
        // Join game room
        player1.join(gameId);
        player2.join(gameId);
        
        // Start game
        this.io.to(gameId).emit('game_start', {
            gameId: gameId,
            players: [
                { id: player1.id, data: player1.playerData, symbol: 'X' },
                { id: player2.id, data: player2.playerData, symbol: 'O' }
            ],
            currentPlayer: player1.id
        });
    }
    
    handleMove(socket, data) {
        const game = this.games.get(socket.gameId);
        if (!game || game.status !== 'playing') return;
        
        const playerIndex = game.players.findIndex(p => p.id === socket.id);
        if (playerIndex !== game.currentPlayer) return;
        
        // Validate move
        if (data.position < 0 || data.position > 8 || game.board[data.position] !== null) {
            return;
        }
        
        // Make move
        game.board[data.position] = playerIndex === 0 ? 'X' : 'O';
        
        // Check for winner
        const winner = this.checkWinner(game.board);
        if (winner) {
            game.status = 'finished';
            this.io.to(game.id).emit('game_end', {
                winner: winner,
                board: game.board,
                winnerPlayer: game.players[playerIndex].id
            });
        } else if (game.board.every(cell => cell !== null)) {
            // Draw
            game.status = 'finished';
            this.io.to(game.id).emit('game_end', {
                winner: 'draw',
                board: game.board
            });
        } else {
            // Continue game
            game.currentPlayer = game.currentPlayer === 0 ? 1 : 0;
            this.io.to(game.id).emit('game_move', {
                position: data.position,
                symbol: playerIndex === 0 ? 'X' : 'O',
                board: game.board,
                nextPlayer: game.players[game.currentPlayer].id
            });
        }
    }
    
    checkWinner(board) {
        const lines = [
            [0, 1, 2], [3, 4, 5], [6, 7, 8], // Rows
            [0, 3, 6], [1, 4, 7], [2, 5, 8], // Columns
            [0, 4, 8], [2, 4, 6] // Diagonals
        ];
        
        for (const [a, b, c] of lines) {
            if (board[a] && board[a] === board[b] && board[a] === board[c]) {
                return board[a];
            }
        }
        return null;
    }
    
    handleDisconnect(socket) {
        // Remove from waiting list
        const waitingIndex = this.waitingPlayers.findIndex(p => p.id === socket.id);
        if (waitingIndex !== -1) {
            this.waitingPlayers.splice(waitingIndex, 1);
        }
        
        // Handle game disconnect
        if (socket.gameId) {
            const game = this.games.get(socket.gameId);
            if (game) {
                this.io.to(socket.gameId).emit('player_disconnected', {
                    playerId: socket.id
                });
                this.games.delete(socket.gameId);
            }
        }
    }
}

// Initialize game server
const gameServer = new GameServer(io);
```

---

## 🔒 Security Considerations

### Authentication with Socket.io
```javascript
const jwt = require('jsonwebtoken');

// Middleware for authentication
io.use((socket, next) => {
    const token = socket.handshake.auth.token;
    
    try {
        const decoded = jwt.verify(token, process.env.JWT_SECRET);
        socket.userId = decoded.id;
        socket.username = decoded.username;
        next();
    } catch (error) {
        next(new Error('Authentication error'));
    }
});

// Rate limiting
const rateLimiter = new Map();

io.use((socket, next) => {
    const userId = socket.userId;
    const now = Date.now();
    
    if (!rateLimiter.has(userId)) {
        rateLimiter.set(userId, []);
    }
    
    const userActions = rateLimiter.get(userId);
    
    // Remove actions older than 1 minute
    const recentActions = userActions.filter(time => now - time < 60000);
    
    if (recentActions.length >= 100) { // Max 100 actions per minute
        next(new Error('Rate limit exceeded'));
        return;
    }
    
    recentActions.push(now);
    rateLimiter.set(userId, recentActions);
    next();
});

// Input validation
socket.on('chat_message', (data) => {
    // Validate input
    if (!data.message || typeof data.message !== 'string') {
        return;
    }
    
    if (data.message.length > 500) {
        socket.emit('error', 'Message too long');
        return;
    }
    
    // Sanitize input
    const sanitizedMessage = data.message
        .replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '')
        .trim();
    
    if (!sanitizedMessage) {
        return;
    }
    
    // Process message
    handleChatMessage(socket, {
        ...data,
        message: sanitizedMessage
    });
});
```

### CORS and Security Headers
```javascript
const io = socketIo(server, {
    cors: {
        origin: process.env.ALLOWED_ORIGINS?.split(',') || ["http://localhost:3000"],
        methods: ["GET", "POST"],
        credentials: true
    },
    allowEIO3: true,
    transports: ['websocket', 'polling']
});

// Additional security middleware
app.use((req, res, next) => {
    res.setHeader('X-Content-Type-Options', 'nosniff');
    res.setHeader('X-Frame-Options', 'DENY');
    res.setHeader('X-XSS-Protection', '1; mode=block');
    next();
});
```

This comprehensive guide covers WebSocket basics, Socket.io implementation, real-time applications, and security considerations. The examples demonstrate practical implementation patterns for building real-time communication features in web applications.
