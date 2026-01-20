<div align="center">

# 🚀 FCA-SAHIL

### Facebook Chat API - Advanced Messenger Bot Framework

[![npm version](https://img.shields.io/npm/v/fca-sahil.svg?style=flat-square)](https://www.npmjs.com/package/fca-sahil)
[![npm downloads](https://img.shields.io/npm/dm/fca-sahil.svg?style=flat-square)](https://www.npmjs.com/package/fca-sahil)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](https://opensource.org/licenses/MIT)
[![Node.js Version](https://img.shields.io/node/v/fca-sahil.svg?style=flat-square)](https://nodejs.org)

**A powerful, reliable, and feature-rich Node.js package for automating Facebook Messenger bots**

[📦 Installation](#-installation) • [🎯 Quick Start](#-quick-start) • [📚 Documentation](#-documentation) • [💡 Examples](#-examples) • [🤝 Support](#-support)

---

### ⚠️ Disclaimer

**Use responsibly!** We are not liable for account bans due to:
- Sending excessive messages (spam)
- Rapid login/logout cycles
- Sharing suspicious URLs
- Violating Facebook's Terms of Service

Always be a responsible Facebook user.

</div>

---

## 📖 Table of Contents

- [🌟 Features](#-features)
- [📦 Installation](#-installation)
- [🎯 Quick Start](#-quick-start)
- [🔧 Configuration](#-configuration)
- [📚 Core Concepts](#-core-concepts)
  - [Login & Authentication](#login--authentication)
  - [Sending Messages](#sending-messages)
  - [Listening to Messages](#listening-to-messages)
  - [Group Management](#group-management)
  - [User Management](#user-management)
- [💡 Examples](#-examples)
- [🔌 API Reference](#-api-reference)
- [❓ FAQ](#-faq)
- [🐛 Troubleshooting](#-troubleshooting)
- [🤝 Contributing](#-contributing)
- [📝 License](#-license)

---

## 🌟 Features

### 🔐 **Robust Authentication**
- ✅ Cookie-based login (JSON or header string format)
- ✅ Automatic session management
- ✅ Auto re-login on connection errors
- ✅ Account lock/suspension detection
- ✅ Token refresh (fb_dtsg) at midnight daily

### 💬 **Advanced Messaging**
- ✅ Send text, images, videos, files, stickers
- ✅ Message reactions and replies
- ✅ Mentions and tagging
- ✅ Edit and unsend messages
- ✅ Forward attachments
- ✅ Typing indicators

### 👥 **Group Management**
- ✅ Create and manage groups
- ✅ Add/remove members
- ✅ Change group name, image, emoji
- ✅ Set nicknames and admin status
- ✅ **Fixed: Support for both 15-digit and 16-digit group UIDs**

### 🎨 **Thread Customization**
- ✅ Change thread colors
- ✅ Set custom emojis
- ✅ Mute/unmute conversations
- ✅ Archive/unarchive threads

### 📊 **Post Interactions**
- ✅ Comment on posts
- ✅ React to posts and stories
- ✅ Create polls
- ✅ Share links and contacts

### ⚡ **Performance & Reliability**
- ✅ MQTT-based real-time messaging
- ✅ Auto-reconnect on disconnect
- ✅ Optimized user agent to reduce logouts
- ✅ Region selection (PRN, PNB, HKG, SYD, VLL, etc.)
- ✅ Proxy support

---

## 📦 Installation

### Prerequisites
- **Node.js**: v10.x or higher
- **npm** or **yarn**

### Install via npm

```bash
npm install fca-sahil@latest
```

### Install via yarn

```bash
yarn add fca-sahil@latest
```

### Verify Installation

```bash
npm list fca-sahil
```

---

## 🎯 Quick Start

### 1️⃣ Basic Echo Bot

Create a simple bot that repeats everything you send:

```javascript
const sahil = require("fca-sahil");

// Your Facebook cookie (see FAQ on how to get it)
const cookie = "your_cookie_here";

sahil.login(cookie, (err, api) => {
  if (err) return console.error(err);
  
  api.listenMqtt((err, event) => {
    if (err) return console.error(err);
    
    if (event.type === "message") {
      api.sendMessage(`You said: ${event.body}`, event.threadID);
    }
  });
});
```

### 2️⃣ Bot with Commands

```javascript
const sahil = require("fca-sahil");
const fs = require("fs");

// Load saved cookie
const cookie = JSON.parse(fs.readFileSync("cookie.json", "utf8"));

sahil.login(cookie, { listenEvents: true }, (err, api) => {
  if (err) return console.error(err);
  
  // Save session
  fs.writeFileSync("cookie.json", JSON.stringify(api.getAppState()));
  
  console.log("✅ Bot is online!");
  
  api.listenMqtt((err, event) => {
    if (err) return console.error(err);
    
    if (event.type === "message") {
      const { body, threadID, senderID } = event;
      
      // Ignore own messages
      if (senderID === api.getCurrentUserID()) return;
      
      // Commands
      switch (body.toLowerCase()) {
        case "/help":
          api.sendMessage("📋 Available commands:\n/help - Show this menu\n/ping - Check bot status\n/info - Get your info", threadID);
          break;
          
        case "/ping":
          api.sendMessage("🏓 Pong! Bot is alive!", threadID);
          break;
          
        case "/info":
          api.getUserInfo(senderID, (err, info) => {
            if (err) return console.error(err);
            api.sendMessage(`👤 Hello ${info[senderID].name}!`, threadID);
          });
          break;
          
        default:
          // Echo other messages
          api.sendMessage(`Echo: ${body}`, threadID);
      }
    }
  });
});
```

---

## 🔧 Configuration

### Available Options

Configure your bot behavior using `setOptions()`:

```javascript
const options = {
  // Listen to your own messages
  selfListen: false,
  
  // Listen to your own events (group changes, etc.)
  selfListenEvent: false,
  
  // Listen to group events (join/leave, name changes)
  listenEvents: true,
  
  // Listen to typing indicators
  listenTyping: false,
  
  // Update online status
  updatePresence: false,
  
  // Force login even if already logged in
  forceLogin: false,
  
  // Automatically mark messages as delivered
  autoMarkDelivery: false,
  
  // Automatically mark messages as read
  autoMarkRead: true,
  
  // Auto reconnect on disconnect
  autoReconnect: true,
  
  // Set online status
  online: true,
  
  // Emit ready event
  emitReady: false,
  
  // Custom user agent
  userAgent: "Mozilla/5.0...",
  
  // Use random user agent (experimental)
  randomUserAgent: false,
  
  // Bypass region (PRN, PNB, HKG, SYD, VLL, LLA, SIN)
  bypassRegion: null,
  
  // Proxy configuration
  proxy: "http://proxy-server:port"
};

sahil.login(cookie, options, (err, api) => {
  // Your bot code
});
```

---

## 📚 Core Concepts

### Login & Authentication

#### 🍪 Getting Your Cookie

**Method 1: Using Cookie Editor Extension**
1. Install a cookie editor extension (Chrome, Firefox, Edge)
2. Log in to Facebook
3. Open the extension and export cookies as JSON

**Method 2: Using Browser DevTools**
1. Open Facebook and log in
2. Press F12 to open DevTools
3. Go to Application → Cookies → facebook.com
4. Copy all cookies

#### 📝 Cookie Formats

**JSON Format:**
```javascript
const cookie = [
  { key: "c_user", value: "100012345678901" },
  { key: "xs", value: "12%3A..." },
  { key: "fr", value: "0abc..." }
  // ... more cookies
];
```

**Header String Format:**
```javascript
const cookie = "c_user=100012345678901; xs=12%3A...; fr=0abc...";
```

#### 💾 Save & Load Sessions

```javascript
const fs = require("fs");
const sahil = require("fca-sahil");

// First login - save session
sahil.login(cookie, (err, api) => {
  if (err) return console.error(err);
  
  // Save app state
  const appState = api.getAppState();
  fs.writeFileSync("session.json", JSON.stringify(appState));
  
  console.log("Session saved!");
});

// Next time - load session
const savedSession = JSON.parse(fs.readFileSync("session.json", "utf8"));

sahil.login(savedSession, (err, api) => {
  if (err) return console.error(err);
  console.log("Logged in with saved session!");
});
```

---

### Sending Messages

#### 📤 Text Messages

```javascript
// Simple text
api.sendMessage("Hello World!", threadID);

// With callback
api.sendMessage("Hello!", threadID, (err, messageInfo) => {
  if (err) return console.error(err);
  console.log("Message sent:", messageInfo.messageID);
});
```

#### 📎 Messages with Attachments

```javascript
const fs = require("fs");

// Single file
api.sendMessage({
  body: "Check out this image!",
  attachment: fs.createReadStream("./photo.jpg")
}, threadID);

// Multiple files
api.sendMessage({
  body: "Multiple files",
  attachment: [
    fs.createReadStream("./image1.jpg"),
    fs.createReadStream("./image2.jpg"),
    fs.createReadStream("./document.pdf")
  ]
}, threadID);
```

#### 🎨 Stickers & Emojis

```javascript
// Send sticker
api.sendMessage({
  sticker: "369239263222822"
}, threadID);

// Send large emoji
api.sendMessage({
  emoji: "👍",
  emojiSize: "large" // small, medium, large
}, threadID);
```

#### 🔗 URLs & Links

```javascript
api.sendMessage({
  url: "https://github.com",
  body: "Check out GitHub!"
}, threadID);
```

#### 👤 Mentions

```javascript
api.sendMessage({
  body: "Hello @John, how are you?",
  mentions: [{
    tag: "@John",
    id: "100012345678901",
    fromIndex: 6 // Position in the message
  }]
}, threadID);
```

#### 💬 Reply to Messages

```javascript
// Reply to a specific message
api.sendMessage("This is a reply", threadID, null, messageID);
```

#### ✏️ Edit Messages

```javascript
api.editMessage("Updated text", messageID, (err) => {
  if (err) return console.error(err);
  console.log("Message edited!");
});
```

#### 🗑️ Unsend Messages

```javascript
api.unsendMessage(messageID, (err) => {
  if (err) return console.error(err);
  console.log("Message unsent!");
});
```

#### 🔄 Forward Attachments

```javascript
api.forwardAttachment(attachmentID, [userID1, userID2], (err) => {
  if (err) return console.error(err);
  console.log("Attachment forwarded!");
});
```

---

### Listening to Messages

#### 👂 Basic Listener

```javascript
api.listenMqtt((err, event) => {
  if (err) return console.error(err);
  
  console.log(event);
});
```

#### 📨 Event Types

```javascript
api.listenMqtt((err, event) => {
  if (err) return console.error(err);
  
  switch (event.type) {
    case "message":
      console.log("New message:", event.body);
      break;
      
    case "message_reply":
      console.log("Reply to message:", event.body);
      break;
      
    case "event":
      console.log("Group event:", event.logMessageType);
      break;
      
    case "typ":
      console.log("Typing indicator:", event.isTyping);
      break;
      
    case "presence":
      console.log("User presence:", event.statuses);
      break;
      
    case "read_receipt":
      console.log("Message read by:", event.reader);
      break;
  }
});
```

#### 🛑 Stop Listening

```javascript
const stopListening = api.listenMqtt((err, event) => {
  if (err) return console.error(err);
  
  if (event.body === "/stop") {
    api.sendMessage("Goodbye!", event.threadID);
    stopListening(); // Stop the listener
  }
});
```

---

### Group Management

#### ➕ Add Users to Group

```javascript
// Add single user
api.addUserToGroup(userID, groupID, (err) => {
  if (err) return console.error(err);
  console.log("User added!");
});

// Add multiple users
api.addUserToGroup([userID1, userID2, userID3], groupID);
```

#### ➖ Remove Users from Group

```javascript
api.removeUserFromGroup(userID, groupID, (err) => {
  if (err) return console.error(err);
  console.log("User removed!");
});
```

#### 🏷️ Change Group Name

```javascript
api.setTitle("New Group Name", groupID, (err) => {
  if (err) return console.error(err);
  console.log("Group name changed!");
});
```

#### 🖼️ Change Group Image

```javascript
const fs = require("fs");

api.changeGroupImage(fs.createReadStream("./group-photo.jpg"), groupID, (err) => {
  if (err) return console.error(err);
  console.log("Group image changed!");
});
```

#### 📝 Change Nicknames

```javascript
api.changeNickname("Cool Nickname", groupID, userID, (err) => {
  if (err) return console.error(err);
  console.log("Nickname changed!");
});
```

#### 👑 Change Admin Status

```javascript
// Make admin
api.changeAdminStatus(groupID, userID, true, (err) => {
  if (err) return console.error(err);
  console.log("User is now admin!");
});

// Remove admin
api.changeAdminStatus(groupID, userID, false);
```

#### 🆕 Create New Group

```javascript
api.createNewGroup([userID1, userID2], "My New Group", (err, groupID) => {
  if (err) return console.error(err);
  console.log("Group created:", groupID);
});
```

#### 🔢 Sending to Groups (15-digit UID Fix)

```javascript
// For groups with 15-digit UIDs, set isGroup to true
const groupID = "123456789012345"; // 15 digits

api.sendMessage("Hello Group!", groupID, null, null, true);

// For 16-digit UIDs, auto-detection works
const groupID16 = "1234567890123456"; // 16 digits
api.sendMessage("Hello Group!", groupID16); // Works automatically
```

---

### User Management

#### 👤 Get User Info

```javascript
// Single user
api.getUserInfo(userID, (err, info) => {
  if (err) return console.error(err);
  
  console.log("Name:", info[userID].name);
  console.log("Profile URL:", info[userID].profileUrl);
  console.log("Gender:", info[userID].gender);
  console.log("Is Friend:", info[userID].isFriend);
});

// Multiple users
api.getUserInfo([userID1, userID2], (err, info) => {
  if (err) return console.error(err);
  console.log(info);
});
```

#### 🔍 Search Users by Name

```javascript
api.getUserID("John Doe", (err, results) => {
  if (err) return console.error(err);
  
  results.forEach(user => {
    console.log("Name:", user.name);
    console.log("ID:", user.userID);
    console.log("Profile:", user.profileUrl);
  });
});
```

#### 👥 Get Friends List

```javascript
api.getFriendsList((err, friends) => {
  if (err) return console.error(err);
  
  friends.forEach(friend => {
    console.log(friend.fullName, "-", friend.userID);
  });
});
```

#### 🆔 Get Current User ID

```javascript
const myUserID = api.getCurrentUserID();
console.log("My ID:", myUserID);
```

#### 🤝 Handle Friend Requests

```javascript
// Accept
api.handleFriendRequest(userID, true, (err) => {
  if (err) return console.error(err);
  console.log("Friend request accepted!");
});

// Decline
api.handleFriendRequest(userID, false);
```

#### 💔 Unfriend User

```javascript
api.unfriend(userID, (err) => {
  if (err) return console.error(err);
  console.log("User unfriended!");
});
```

#### 🚫 Block/Unblock User

```javascript
// Block
api.changeBlockedStatus(userID, true, (err) => {
  if (err) return console.error(err);
  console.log("User blocked!");
});

// Unblock
api.changeBlockedStatus(userID, false);
```

---

## 💡 Examples

### 🤖 Auto-Reply Bot

```javascript
const sahil = require("fca-sahil");
const fs = require("fs");

const cookie = JSON.parse(fs.readFileSync("cookie.json", "utf8"));

sahil.login(cookie, { listenEvents: true }, (err, api) => {
  if (err) return console.error(err);
  
  const replies = {
    "hi": "Hello! How can I help you?",
    "hello": "Hi there! 👋",
    "bye": "Goodbye! See you later! 👋",
    "help": "I'm a simple auto-reply bot. Say 'hi', 'hello', or 'bye'!"
  };
  
  api.listenMqtt((err, event) => {
    if (err) return console.error(err);
    
    if (event.type === "message") {
      const message = event.body.toLowerCase();
      const reply = replies[message];
      
      if (reply) {
        api.sendMessage(reply, event.threadID);
      }
    }
  });
});
```

### 📊 Group Statistics Bot

```javascript
api.listenMqtt((err, event) => {
  if (err) return console.error(err);
  
  if (event.type === "message" && event.body === "/stats") {
    api.getThreadInfo(event.threadID, (err, info) => {
      if (err) return console.error(err);
      
      const stats = `
📊 Group Statistics
━━━━━━━━━━━━━━━━━
👥 Members: ${info.participantIDs.length}
💬 Total Messages: ${info.messageCount}
📬 Unread: ${info.unreadCount}
📅 Created: ${new Date(info.timestamp).toLocaleDateString()}
      `;
      
      api.sendMessage(stats, event.threadID);
    });
  }
});
```

### 🎲 Random Game Bot

```javascript
api.listenMqtt((err, event) => {
  if (err) return console.error(err);
  
  if (event.type === "message") {
    const { body, threadID } = event;
    
    if (body === "/roll") {
      const dice = Math.floor(Math.random() * 6) + 1;
      api.sendMessage(`🎲 You rolled a ${dice}!`, threadID);
    }
    
    if (body === "/flip") {
      const coin = Math.random() < 0.5 ? "Heads" : "Tails";
      api.sendMessage(`🪙 ${coin}!`, threadID);
    }
  }
});
```

### 📸 Image Sender Bot

```javascript
const fs = require("fs");

api.listenMqtt((err, event) => {
  if (err) return console.error(err);
  
  if (event.type === "message" && event.body === "/cat") {
    api.sendMessage({
      body: "Here's a cute cat! 🐱",
      attachment: fs.createReadStream("./cat.jpg")
    }, event.threadID);
  }
});
```

---

## 🔌 API Reference

### Core Functions

| Function | Description |
|----------|-------------|
| `login(cookie, options, callback)` | Login to Facebook |
| `setOptions(options)` | Configure bot behavior |
| `getAppState()` | Get current session state |

### Messaging Functions

| Function | Description |
|----------|-------------|
| `sendMessage(message, threadID, callback, messageID, isGroup)` | Send messages |
| `sendMessageMqtt(message, threadID, callback, messageID)` | Send via MQTT |
| `editMessage(text, messageID, callback)` | Edit sent messages |
| `unsendMessage(messageID, callback)` | Unsend messages |
| `forwardAttachment(attachmentID, userOrGroupIDs, callback)` | Forward attachments |
| `setMessageReaction(reaction, messageID, callback)` | React to messages |
| `sendTypingIndicator(threadID, callback)` | Show typing indicator |

### Group Management Functions

| Function | Description |
|----------|-------------|
| `addUserToGroup(userID, threadID, callback)` | Add members |
| `removeUserFromGroup(userID, threadID, callback)` | Remove members |
| `changeNickname(nickname, threadID, participantID, callback)` | Change nicknames |
| `setTitle(title, threadID, callback)` | Change group name |
| `changeGroupImage(attachment, threadID, callback)` | Change group image |
| `changeAdminStatus(threadID, adminIDs, adminStatus, callback)` | Change admin status |
| `createNewGroup(participantIDs, groupTitle, callback)` | Create new group |

### User Information Functions

| Function | Description |
|----------|-------------|
| `getUserInfo(userID, callback)` | Get user details |
| `getUserID(name, callback)` | Get user ID by name |
| `getFriendsList(callback)` | Get friends list |
| `getCurrentUserID()` | Get current bot's user ID |
| `handleFriendRequest(userID, accept, callback)` | Accept/decline friend request |
| `unfriend(userID, callback)` | Unfriend user |
| `changeBlockedStatus(userID, block, callback)` | Block/unblock user |

### Thread Functions

| Function | Description |
|----------|-------------|
| `getThreadInfo(threadID, callback)` | Get thread details |
| `getThreadHistory(threadID, amount, timestamp, callback)` | Get message history |
| `getThreadList(limit, timestamp, tags, callback)` | Get thread list |
| `searchForThread(name, callback)` | Search threads by name |
| `changeThreadColor(color, threadID, callback)` | Change thread color |
| `changeThreadEmoji(emoji, threadID, callback)` | Change thread emoji |
| `muteThread(threadID, muteSeconds, callback)` | Mute/unmute thread |
| `changeArchivedStatus(threadID, archive, callback)` | Archive/unarchive thread |
| `deleteThread(threadID, callback)` | Delete thread |

### Post & Comment Functions

| Function | Description |
|----------|-------------|
| `createCommentPost(message, postID, callback, replyCommentID)` | Comment on posts |
| `setPostReaction(reaction, postID, callback)` | React to posts |
| `createPost(post, callback)` | Create new post |
| `createPoll(poll, threadID, callback)` | Create poll |

### Status Functions

| Function | Description |
|----------|-------------|
| `markAsRead(threadID, callback)` | Mark as read |
| `markAsDelivered(messageID, threadID, callback)` | Mark as delivered |
| `markAsReadAll(callback)` | Mark all as read |

### Listening Functions

| Function | Description |
|----------|-------------|
| `listenMqtt(callback)` | Listen for messages and events |
| `stopListenMqtt()` | Stop listening |

---

## ❓ FAQ

### Q1: How do I get my Facebook cookie?

**Answer:** Use a cookie editor extension:
1. Install "EditThisCookie" (Chrome) or "Cookie-Editor" (Firefox)
2. Log in to Facebook
3. Click the extension icon
4. Export cookies as JSON

### Q2: Why is my bot not working with 15-digit group IDs?

**Answer:** This has been fixed! Use the `isGroup` parameter:

```javascript
api.sendMessage("Hello!", "123456789012345", null, null, true);
```

### Q3: How do I handle errors?

**Answer:** Always use callbacks or try-catch:

```javascript
api.sendMessage("Hello!", threadID, (err, info) => {
  if (err) {
    console.error("Error:", err);
    return;
  }
  console.log("Success:", info);
});
```

### Q4: Can I use this for multiple accounts?

**Answer:** Yes! Create separate instances:

```javascript
sahil.login(cookie1, (err, api1) => {
  // Bot 1
});

sahil.login(cookie2, (err, api2) => {
  // Bot 2
});
```

### Q5: How do I prevent my account from being banned?

**Answer:**
- Don't send too many messages too quickly
- Use realistic delays between actions
- Don't spam users
- Follow Facebook's Terms of Service

### Q6: What are the available message reactions?

**Answer:** 👍 ❤️ 😆 😮 😢 😠 👎

```javascript
api.setMessageReaction("❤️", messageID);
```

### Q7: How do I save and restore sessions?

**Answer:**

```javascript
// Save
const appState = api.getAppState();
fs.writeFileSync("session.json", JSON.stringify(appState));

// Load
const session = JSON.parse(fs.readFileSync("session.json"));
sahil.login(session, callback);
```

---

## 🐛 Troubleshooting

### Error: "Not logged in"

**Solution:** Your cookie has expired. Get a new cookie and login again.

### Error: "Error 1545012"

**Solution:** You're not a member of the group. Make sure the bot account is in the group.

### Error: "Checkpoint detected"

**Solution:** Facebook requires verification. Log in with a browser and complete the checkpoint.

### Bot keeps disconnecting

**Solution:** Enable auto-reconnect:

```javascript
sahil.login(cookie, { autoReconnect: true }, callback);
```

### Messages not sending to groups

**Solution:** For 15-digit group IDs, use the `isGroup` parameter:

```javascript
api.sendMessage("Hello!", groupID, null, null, true);
```

---

## 🤝 Contributing

We welcome contributions! Here's how you can help:

1. 🐛 Report bugs
2. 💡 Suggest new features
3. 📝 Improve documentation
4. 🔧 Submit pull requests

## 📝 License

This project is licensed under the **MIT License**.

```
MIT License

Copyright (c) 2026 Sahil Ansari 

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---
---

## 🙏 Acknowledgments

- Thanks to all contributors and users
- Special thanks to the Facebook Chat API community
- Built with ❤️ by **Sahil**

---

## 📞 Support

Need help? Here's how to get support:

1. 📖 Check the [Documentation](./DOCUMENTATION.md)
2. 🔍 Search [existing issues](https://github.com/Sahil09/fca-sahil/issues)
3. 💬 Open a [new issue](https://github.com/Sahil09/fca-sahil/issues/new)
4. 📧 Contact the developer

---

<div align="center">

### ⭐ Star this repository if you find it helpful!

**Made with ❤️ by Sahil**

[⬆ Back to Top](#-fca-sahil)


</div>


