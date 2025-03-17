# Basic Redis Chat App Demo (Flask, Socket.IO, Redis)

This project showcases how to build a **real-time chat application** using **Python (Flask)**, **Socket.IO**, and **Redis**. It leverages **Redis Pub/Sub** and **WebSockets** for seamless message communication between the client and server.

---

## 📸 Screenshots

<a href="https://github.com/redis-developer/basic-redis-chat-app-demo-python/raw/master/docs/screenshot000.png">
  <img src="https://github.com/redis-developer/basic-redis-chat-app-demo-python/raw/master/docs/screenshot000.png" width="49%">
</a>
<a href="https://github.com/redis-developer/basic-redis-chat-app-demo-python/raw/master/docs/screenshot001.png">
  <img src="https://github.com/redis-developer/basic-redis-chat-app-demo-python/raw/master/docs/screenshot001.png" width="49%">
</a>

---

## 🎥 Overview Video

Watch this short video to understand how the project works:

[![Watch the video on YouTube](https://github.com/redis-developer/basic-redis-chat-app-demo-python/raw/master/docs/YTThumbnail.png)](https://www.youtube.com/watch?v=miK7xDkDXF0)

---

## 🛠️ Tech Stack

- **Frontend:** React, Socket.IO
- **Backend:** Flask, Redis, Flask-SocketIO

---

## 📌 How It Works?

### 1️⃣ Initialization
- On startup, the **Redis database** is initialized with demo users and rooms if they don't exist.
- Users are **stored in Redis hashes**, and each user is added to the **"General" room**.
- Private message rooms are created dynamically.

### 2️⃣ User Registration & Authentication
- Users are stored using Redis Hash (`HSET user:{id} username "nick" password "hashed_pw"`)
- Each user gets a unique ID by incrementing `total_users`.
- A username lookup key is also created (`SET username:nick user:1`).

### 3️⃣ Message Storage & Real-time Updates
- **Messages are stored in Redis Sorted Sets** (`ZADD room:{roomId} {timestamp} {message}`)
- **Redis Pub/Sub is used to broadcast messages** (`PUBLISH MESSAGES {message_data}`)

### 4️⃣ Room Management
- **Public rooms** have a name (`SET room:{roomId}:name "General"`)
- **Private rooms** are created dynamically between two users (`SADD user:1:rooms 1:2`).
- Messages in private rooms are **only visible to those two users**.

---

## 🔹 Code Examples

### ✅ Creating a New User in Redis

```python
def create_user(username, password):
    hashed_password = bcrypt.hashpw(password.encode("utf-8"), bcrypt.gensalt(10))
    next_id = redis_client.incr("total_users")
    user_key = f"user:{next_id}"
    
    redis_client.set(f"username:{username}", user_key)
    redis_client.hmset(user_key, {"username": username, "password": hashed_password})
    redis_client.sadd(f"user:{next_id}:rooms", "0")  # Add to General Room
    
    return {"id": next_id, "username": username}
```

### ✅ Storing & Retrieving Messages from Redis

```python
def store_message(room_id, message):
    message_string = json.dumps(message)
    redis_client.zadd(f"room:{room_id}", {message_string: int(message["date"])})

def get_messages(room_id, offset=0, size=50):
    messages = redis_client.zrevrange(f"room:{room_id}", offset, offset + size)
    return [json.loads(msg.decode("utf-8")) for msg in messages]
```

---

## 📡 Redis Pub/Sub for Real-time Communication

Each message is published to Redis:

```python
redis_client.publish("MESSAGES", json.dumps({"serverId": SERVER_ID, "type": "message", "data": message}))
```

And the chat app listens for new messages:

```python
pubsub = redis_client.pubsub(ignore_subscribe_messages=True)
pubsub.subscribe("MESSAGES")
for message in pubsub.listen():
    handle_new_message(json.loads(message["data"]))
```

---

## 🚀 How to Run Locally?

### 🔹 1️⃣ Set Up Environment Variables
Copy `.env.sample` to create `.env`, then update:

```ini
REDIS_ENDPOINT_URI=127.0.0.1:6379
REDIS_PASSWORD=your_redis_password
SECRET_KEY=your_secret_key
```

### 🔹 2️⃣ Start Redis Server
Redis must be running before starting the app:

```sh
redis-server
```

OR (for Windows WSL):

```sh
wsl redis-server
```

### 🔹 3️⃣ Run the Frontend

```sh
cd client
yarn install
yarn start
```

Access the chat UI at: **http://localhost:3000**

### 🔹 4️⃣ Run the Backend

```sh
python3 -m venv venv  # Create virtual environment (only first time)
source venv/bin/activate  # Activate virtual environment
pip3 install -r requirements.txt  # Install dependencies
python3 app.py  # Start Flask server
```

Backend will run at: **http://localhost:5000**
