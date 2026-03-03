<!DOCTYPE html>
<html>
<head>
    <title>My Social App</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>

<h2>Social Media Feed</h2>

<textarea id="postContent" placeholder="What's on your mind?"></textarea>
<button onclick="createPost()">Post</button>

<div id="posts"></div>

<hr>
<a href="chat.html">Go to Chat</a>

<script src="app.js"></script>
</body>
</html><!DOCTYPE html>
<html>
<head>
    <title>Chat</title>
</head>
<body>

<h2>Live Chat</h2>

<div id="messages"></div>
<input id="messageInput" placeholder="Type message">
<button onclick="sendMessage()">Send</button>

<script src="/socket.io/socket.io.js"></script>
<script>
const socket = io();

function sendMessage(){
    const msg = document.getElementById("messageInput").value;
    socket.emit("sendMessage", msg);
}

socket.on("receiveMessage", (msg) => {
    const div = document.createElement("div");
    div.innerText = msg;
    document.getElementById("messages").appendChild(div);
});
</script>

</body>
</html>async function createPost(){
    const content = document.getElementById("postContent").value;

    await fetch("/post", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ user: "User1", content })
    });

    loadPosts();
}

async function loadPosts(){
    const res = await fetch("/posts");
    const posts = await res.json();

    const container = document.getElementById("posts");
    container.innerHTML = "";

    posts.forEach(post => {
        const div = document.createElement("div");
        div.innerHTML = `<b>${post.user}</b>: ${post.content}`;
        container.appendChild(div);
    });
}

loadPosts();npm init -y
npm install express mongoose socket.io corssocial-app/
│
├── server.js
├── package.json
├── models/
│     └── User.js
│     └── Post.js
├── public/
│     ├── index.html
│     ├── chat.html
│     ├── style.css
│     └── app.jsconst express = require("express");
const mongoose = require("mongoose");
const http = require("http");
const socketIo = require("socket.io");
const cors = require("cors");

const app = express();
const server = http.createServer(app);
const io = socketIo(server);

app.use(cors());
app.use(express.json());
app.use(express.static("public"));

mongoose.connect("mongodb://localhost:27017/socialapp")
.then(() => console.log("MongoDB Connected"));

/* =====================
   MODELS
===================== */

const User = mongoose.model("User", {
    username: String,
    email: String,
    password: String
});

const Post = mongoose.model("Post", {
    user: String,
    content: String,
    date: { type: Date, default: Date.now }
});

/* =====================
   ROUTES
===================== */

// Create Post
app.post("/post", async (req, res) => {
    const post = new Post(req.body);
    await post.save();
    res.json(post);
});

// Get Posts
app.get("/posts", async (req, res) => {
    const posts = await Post.find().sort({ date: -1 });
    res.json(posts);
});

/* =====================
   REAL-TIME CHAT
===================== */

io.on("connection", (socket) => {
    console.log("User connected");

    socket.on("sendMessage", (data) => {
        io.emit("receiveMessage", data);
    });

    socket.on("disconnect", () => {
        console.log("User disconnected");
    });
});

server.listen(5000, () => {
    console.log("Server running on port 5000");
});
