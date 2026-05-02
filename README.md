git init
git add .
git commit -m "Initial MERN blog app"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/mern-blog-app.git
git push -u origin main

# MERN Blog Application

Full-stack blog platform built using MongoDB, Express, React, and Node.js.

## Features
- User authentication (JWT)
- Create, read, delete blog posts
- RESTful API
- Responsive UI

## Tech Stack
- Frontend: React, Axios
- Backend: Node.js, Express
- Database: MongoDB (Mongoose)
- Auth: JWT + bcrypt

## Setup

### Backend
cd server  
npm install  
create .env file  
npm start  

### Frontend
cd client  
npm install  
npm start  

## Deployment
- Frontend: Vercel
- Backend: Render
- Database: MongoDB Atlas

## Future Improvements
- Edit posts
- Comments
- User profiles
- Dark mode
mern-blog-app/
│
├── client/        # React frontend
├── server/        # Node/Express backend
├── README.md
└── .gitignore
Backend (server)
📁 /server/app.js
const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");
const jwt = require("jsonwebtoken");
const bcrypt = require("bcrypt");

const app = express();
app.use(express.json());
app.use(cors());

mongoose.connect(process.env.MONGO_URI);

// Models
const User = mongoose.model("User", {
  email: String,
  password: String
});

const Post = mongoose.model("Post", {
  title: String,
  content: String,
  user: String
});

// Middleware
const auth = (req, res, next) => {
  const token = req.headers.authorization;
  if (!token) return res.status(401).send("No token");

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.userId = decoded.id;
    next();
  } catch {
    res.status(401).send("Invalid token");
  }
};

// Routes

// Register
app.post("/api/register", async (req, res) => {
  const hashed = await bcrypt.hash(req.body.password, 10);
  const user = await User.create({
    email: req.body.email,
    password: hashed
  });
  res.json(user);
});

// Login
app.post("/api/login", async (req, res) => {
  const user = await User.findOne({ email: req.body.email });
  if (!user) return res.status(404).send("User not found");

  const match = await bcrypt.compare(req.body.password, user.password);
  if (!match) return res.status(401).send("Wrong password");

  const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET);
  res.json({ token });
});

// Get posts
app.get("/api/posts", async (req, res) => {
  const posts = await Post.find();
  res.json(posts);
});

// Create post
app.post("/api/posts", auth, async (req, res) => {
  const post = await Post.create({
    title: req.body.title,
    content: req.body.content,
    user: req.userId
  });
  res.json(post);
});

// Delete post
app.delete("/api/posts/:id", auth, async (req, res) => {
  await Post.findByIdAndDelete(req.params.id);
  res.sendStatus(200);
});

app.listen(3000, () => console.log("Server running"));
/server/.env
MONGO_URI=your_mongodb_connection
JWT_SECRET=supersecretkey
/server/package.json
{
  "name": "server",
  "version": "1.0.0",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  }
}
Frontend
/client/src/App.js
import { useState, useEffect } from "react";
import axios from "axios";

const API = process.env.REACT_APP_API_URL;

function App() {
  const [posts, setPosts] = useState([]);
  const [form, setForm] = useState({ title: "", content: "" });

  useEffect(() => {
    fetchPosts();
  }, []);

  const fetchPosts = async () => {
    const res = await axios.get(`${API}/api/posts`);
    setPosts(res.data);
  };

  const createPost = async () => {
    await axios.post(`${API}/api/posts`, form, {
      headers: {
        Authorization: localStorage.getItem("token")
      }
    });
    fetchPosts();
  };

  return (
    <div style={{ maxWidth: 700, margin: "auto" }}>
      <h1>Blog</h1>

      <input
        placeholder="Title"
        onChange={e => setForm({ ...form, title: e.target.value })}
      />
      <textarea
        placeholder="Content"
        onChange={e => setForm({ ...form, content: e.target.value })}
      />
      <button onClick={createPost}>Add</button>

      {posts.map(p => (
        <div key={p._id}>
          <h3>{p.title}</h3>
          <p>{p.content}</p>
        </div>
      ))}
    </div>
  );
}

export default App;
/client/.env


