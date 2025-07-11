# Realtime-comm
const express = require('express');
const http = require('http');
const { Server } = require('socket.io');
const cors = require('cors');

const app = express();
app.use(cors());

const server = http.createServer(app);
const io = new Server(server, {
  cors: { origin: '*' }
});

// Handle socket events
io.on('connection', (socket) => {
  console.log('User connected:', socket.id);

  socket.on('join_room', (room) => {
    socket.join(room);
  });

  socket.on('send_message', (data) => {
    io.to(data.room).emit('receive_message', data);
  });

  socket.on('typing', (room) => {
    socket.to(room).emit('user_typing');
  });

  socket.on('disconnect', () => {
    console.log('User disconnected:', socket.id);
  });
});

server.listen(3001, () => {
  console.log('Server running on port 3001');
});

cd client
npm install socket.io-client
import { io } from 'socket.io-client';
const socket = io('http://localhost:3001');
export default socket;
import React, { useState, useEffect } from 'react';
import socket from '../socket';

const Chat = ({ username, room }) => {
  const [message, setMessage] = useState('');
  const [messages, setMessages] = useState([]);
  const [isTyping, setIsTyping] = useState(false);

  useEffect(() => {
    socket.emit('join_room', room);

    socket.on('receive_message', (data) => {
      setMessages((prev) => [...prev, data]);
    });

    socket.on('user_typing', () => {
      setIsTyping(true);
      setTimeout(() => setIsTyping(false), 1000);
    });
  }, [room]);

  const sendMessage = () => {
    if (message.trim()) {
      const data = { room, author: username, message, time: new Date().toLocaleTimeString() };
      socket.emit('send_message', data);
      setMessages((prev) => [...prev, data]);
      setMessage('');
    }
  };

  const handleTyping = () => {
    socket.emit('typing', room);
  };

  return (
    <div>
      <div>
        {messages.map((msg, i) => (
          <div key={i}><b>{msg.author}</b>: {msg.message} <i>({msg.time})</i></div>
        ))}
      </div>
      {isTyping && <p>Someone is typing...</p>}
      <input value={message} onChange={(e) => setMessage(e.target.value)} onKeyDown={handleTyping} />
      <button onClick={sendMessage}>Send</button>
    </div>
  );
};

export default Chat;
# In server/
node server.js

# In client/
npm start
