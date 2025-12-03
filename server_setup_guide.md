# How to Add Your Own Server to P2P Chat

## Quick Setup Options

### Option 1: Glitch.com (Easiest - 5 minutes)

#### Step 1: Create Account
1. Go to **https://glitch.com**
2. Click "Sign in" → Use GitHub or Email
3. Free account is perfect!

#### Step 2: Create Project
1. Click "New Project"
2. Select "glitch-hello-node"
3. Wait for it to load

#### Step 3: Add Server Code
1. **Delete existing files:**
   - Click `server.js` → Delete
   - Click `public` folder → Delete all contents
   - Keep `package.json`

2. **Create new `server.js`:**
   - Click "+ New File" → name it `server.js`
   - Paste this code:

```javascript
const WebSocket = require('ws');
const http = require('http');

const PORT = process.env.PORT || 3000;
const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('P2P Chat Signaling Server Running');
});

const wss = new WebSocket.Server({ server });
const rooms = new Map();
const clients = new Map();

wss.on('connection', (ws) => {
  let clientId = null;
  let currentRoom = null;
  
  ws.on('message', (message) => {
    try {
      const data = JSON.parse(message);
      
      switch (data.type) {
        case 'register':
          clientId = data.clientId;
          clients.set(clientId, ws);
          console.log(`Client registered: ${clientId}`);
          break;
          
        case 'create-room':
          const roomId = data.roomId;
          rooms.set(roomId, {
            name: data.roomName,
            isPrivate: data.isPrivate,
            creator: clientId,
            members: new Set([clientId]),
            key: data.encryptionKey,
            created: Date.now()
          });
          currentRoom = roomId;
          
          ws.send(JSON.stringify({
            type: 'room-created',
            roomId: roomId
          }));
          
          if (!data.isPrivate) broadcastRoomList();
          break;
          
        case 'join-room':
          const targetRoom = rooms.get(data.roomId);
          if (!targetRoom) {
            ws.send(JSON.stringify({ type: 'error', message: 'Room not found' }));
            break;
          }
          
          targetRoom.members.add(clientId);
          currentRoom = data.roomId;
          
          const membersList = Array.from(targetRoom.members).filter(id => id !== clientId);
          ws.send(JSON.stringify({
            type: 'room-joined',
            roomId: data.roomId,
            roomName: targetRoom.name,
            encryptionKey: targetRoom.key,
            members: membersList
          }));
          
          broadcastToRoom(data.roomId, {
            type: 'peer-joined',
            peerId: clientId
          }, clientId);
          
          broadcastRoomList();
          break;
          
        case 'leave-room':
          if (currentRoom && rooms.has(currentRoom)) {
            const room = rooms.get(currentRoom);
            room.members.delete(clientId);
            
            broadcastToRoom(currentRoom, {
              type: 'peer-left',
              peerId: clientId
            }, clientId);
            
            if (room.members.size === 0) rooms.delete(currentRoom);
            currentRoom = null;
            broadcastRoomList();
          }
          break;
          
        case 'signal':
          const targetClient = clients.get(data.targetId);
          if (targetClient && targetClient.readyState === WebSocket.OPEN) {
            targetClient.send(JSON.stringify({
              type: 'signal',
              fromId: clientId,
              signal: data.signal
            }));
          }
          break;
          
        case 'get-rooms':
          sendRoomList(ws);
          break;
      }
    } catch (error) {
      console.error('Error:', error);
    }
  });
  
  ws.on('close', () => {
    if (currentRoom && rooms.has(currentRoom)) {
      const room = rooms.get(currentRoom);
      room.members.delete(clientId);
      
      broadcastToRoom(currentRoom, {
        type: 'peer-left',
        peerId: clientId
      }, clientId);
      
      if (room.members.size === 0) rooms.delete(currentRoom);
      broadcastRoomList();
    }
    clients.delete(clientId);
  });
});

function broadcastToRoom(roomId, message, excludeId = null) {
  const room = rooms.get(roomId);
  if (!room) return;
  
  room.members.forEach(memberId => {
    if (memberId !== excludeId) {
      const client = clients.get(memberId);
      if (client && client.readyState === WebSocket.OPEN) {
        client.send(JSON.stringify(message));
      }
    }
  });
}

function sendRoomList(ws) {
  const publicRooms = Array.from(rooms.entries())
    .filter(([_, room]) => !room.isPrivate)
    .map(([id, room]) => ({
      id: id,
      name: room.name,
      users: room.members.size,
      created: room.created
    }));
  
  ws.send(JSON.stringify({
    type: 'room-list',
    rooms: publicRooms
  }));
}

function broadcastRoomList() {
  const publicRooms = Array.from(rooms.entries())
    .filter(([_, room]) => !room.isPrivate)
    .map(([id, room]) => ({
      id: id,
      name: room.name,
      users: room.members.size,
      created: room.created
    }));
  
  const message = JSON.stringify({
    type: 'room-list',
    rooms: publicRooms
  });
  
  clients.forEach(client => {
    if (client.readyState === WebSocket.OPEN) {
      client.send(message);
    }
  });
}

server.listen(PORT, () => {
  console.log(`Signaling server running on port ${PORT}`);
});
```

3. **Update `package.json`:**
   - Click on `package.json`
   - Replace everything with:

```json
{
  "name": "p2p-chat-server",
  "version": "1.0.0",
  "description": "P2P Chat Signaling Server",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "ws": "^8.14.2"
  },
  "engines": {
    "node": "16.x"
  }
}
```

#### Step 4: Get Your Server URL
1. Glitch will auto-start your server
2. Click "Share" → "Live App"
3. Your URL will be: `https://YOUR-PROJECT-NAME.glitch.me`
4. **Important:** Change `https://` to `wss://` for the chat
5. Your server URL is: `wss://YOUR-PROJECT-NAME.glitch.me`

#### Step 5: Use in Chat
1. Open the P2P chat HTML file
2. Click "SERVER MODE" tab
3. Paste: `wss://YOUR-PROJECT-NAME.glitch.me`
4. Click "CONNECT TO SERVER"
5. Create rooms and share the URL with friends!

---

### Option 2: Replit (Also Easy)

1. Go to **https://replit.com**
2. Create "Node.js" repl
3. Paste the same `server.js` code above
4. Create `package.json` with the same content
5. Click "Run"
6. Your URL: `wss://YOUR-REPL-NAME.YOUR-USERNAME.repl.co`

---

### Option 3: Your Own Computer (For Testing)

#### Requirements:
- Node.js installed
- Terminal/Command Prompt

#### Steps:
1. Create folder: `mkdir p2p-server`
2. Go to folder: `cd p2p-server`
3. Initialize: `npm init -y`
4. Install WebSocket: `npm install ws`
5. Create `server.js` with code above
6. Run: `node server.js`
7. Use URL: `ws://localhost:3000`

**Note:** Only works on your local network!

---

## How Users Connect to Your Server

Once your server is running:

1. **Share the URL** with anyone who wants to join
2. They open the P2P chat HTML file
3. They click "SERVER MODE"
4. Paste your server URL (e.g., `wss://mychat.glitch.me`)
5. Click "CONNECT TO SERVER"
6. They can now see and join rooms!

---

## Server Features

✅ **Rooms** - Create public chat rooms  
✅ **Multiple Users** - Many people can join one room  
✅ **Peer Discovery** - Automatically connects users in same room  
✅ **No Message Storage** - Server only coordinates, doesn't see messages  
✅ **Free Hosting** - Glitch/Replit free tiers work great  
✅ **Auto-Restart** - Glitch restarts your server if it goes down  

---

## Troubleshooting

**"Connection failed"**
- Make sure URL starts with `wss://` (not `https://`)
- Check that server is running (visit the URL in browser)
- Try adding the port if needed: `wss://yourserver.com:3000`

**"Server keeps sleeping"**
- Glitch free tier sleeps after 5 minutes of inactivity
- Upgrade to Glitch Pro ($8/month) for always-on
- Or use a service like UptimeRobot to ping it every 5 min

**"Can't create rooms"**
- Refresh the page
- Check browser console for errors (F12)
- Make sure server code is correct

---

## Advanced: Custom Domain

Want `wss://chat.yourdomain.com`?

1. Deploy to a service that supports custom domains:
   - Railway.app (free tier)
   - Render.com (free tier)
   - Your own VPS (DigitalOcean, Linode, etc.)

2. Add your domain in the service settings
3. Enable SSL (usually automatic)
4. Use `wss://chat.yourdomain.com`

---

## Cost Summary

| Service | Free Tier | Always On? | Custom Domain? |
|---------|-----------|------------|----------------|
| Glitch | ✅ Yes | ❌ Sleeps after 5min | ❌ No |
| Replit | ✅ Yes | ❌ Sleeps when idle | ✅ Yes (paid) |
| Railway | ✅ Yes ($5 credit) | ✅ Yes | ✅ Yes |
| Render | ✅ Yes | ❌ Spins down after 15min | ✅ Yes |

**Recommendation:** Start with Glitch for testing, upgrade to Railway if you need 24/7 uptime.

---

## Security Notes

- Server only coordinates connections, never sees messages
- All messages are encrypted end-to-end
- Server logs connection/disconnection events (not message content)
- Use HTTPS (wss://) in production for extra security
- Don't run the server on important machines (use cloud hosting)

---

## Need Help?

Common issues:
1. **Wrong protocol** → Use `wss://` not `https://`
2. **Server sleeping** → Glitch sleeps after 5min, refresh to wake
3. **Port blocked** → Some networks block WebSocket, try different WiFi
4. **Code errors** → Check you copied the FULL server code

The server is just 200 lines of code - it only helps peers find each other, then gets out of the way! 🚀