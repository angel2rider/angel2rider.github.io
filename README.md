# 📸 Pichost: Infinite Cloud Storage
**Powered by sheer audacity, Cloudflare Workers, and Telegram's server bills.**

![License](https://img.shields.io/badge/license-MIT-blue)
![Architecture](https://img.shields.io/badge/architecture-Serverless-orange)
![Storage Cost](https://img.shields.io/badge/storage%20cost-$0.00-brightgreen)

Welcome to **Pichost**. Are you tired of AWS S3 bleeding your wallet dry? Does the thought of paying for Google Cloud Storage keep you up at night? Have you ever looked at Telegram and thought, *"Wow, that's a really nice, unlimited, and free Content Delivery Network"*? 

If so, you are in the exact right place.

Pichost is a modern, responsive, and blazing-fast image hosting platform. It uses **Cloudflare Workers** as the backend logic, **Cloudflare KV** for the database, and a **Telegram Bot** as an infinite, free storage bucket.

---

## 🚨 THE GREAT PRIVACY WARNING 🚨
When you first open `index.html`, it is pre-configured to use **my** default backend (`https://pichost.vivekpereiraalbert.workers.dev/`) so you can test it out instantly. 

**You are completely free to use it, BUT:** 
If you use my backend, **I (TheGT) can technically see the images you upload** because they are going into my Telegram channel. I am not Mark Zuckerberg, but I highly recommend you **deploy your own backend** (it takes 10 minutes) if you want total privacy and control over your data. You can easily switch the backend URL in the "Settings" tab of the app!

---

## 🧠 How Does This Black Magic Work?

We essentially use Telegram as a headless CMS/Bucket. Here is the architecture:

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant Browser as Pichost Frontend
    participant Worker as Cloudflare Worker (API)
    participant KV as Cloudflare KV (Database)
    participant Telegram as Telegram Bot API

    User->>Browser: Drags & Drops Image
    Browser->>Worker: POST /upload (with Session Cookie)
    Worker->>Telegram: sendPhoto(chat_id, image_bytes)
    Telegram-->>Worker: Returns { file_id, message_id }
    Worker->>KV: put("img:1234", { file_id, owner })
    Worker-->>Browser: 200 OK - Image Uploaded!
    
    Note over User, Telegram: --- Viewing the Image ---
    
    Browser->>Worker: GET /raw/1234
    Worker->>KV: get("img:1234")
    KV-->>Worker: Returns file_id
    Worker->>Telegram: getFile(file_id)
    Telegram-->>Worker: Returns Temporary Download URL
    Worker->>Telegram: Fetch Image Stream
    Telegram-->>Worker: Image Bytes
    Worker-->>Browser: Streams image directly to user (with Edge Caching!)
