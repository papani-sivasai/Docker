# Containerising a Three-Tier MERN E-Commerce Application with Docker and Nginx

> Build and deploy a complete MERN stack e-commerce app using Docker, Nginx, and Docker Compose.

So far in this series, we’ve learned how to build Docker images, run containers, manage networks, use volumes, and simplify setups with Docker Compose.

Now, let’s put it all together into a **real-world, three-tier e-commerce MERN application** using **Docker + Nginx**.

---

## What You’ll Learn

* How to containerize a **MERN (MongoDB, Express, React, Node.js)** app
* Writing separate **Dockerfiles** for frontend and backend
* How Docker Compose connects multi-container apps
* Why you can’t directly access containerized apps via localhost
* How **Nginx** solves this using reverse proxy and static serving

---

## Application Overview

This e-commerce app consists of four layers:

| Layer        | Technology                    | Purpose                        |
| ------------ | ----------------------------- | ------------------------------ |
| **Frontend** | React + Vite (`client/`)      | Customer UI                    |
| **Backend**  | Node.js + Express (`server/`) | Business logic and API         |
| **Database** | MongoDB Atlas (Cloud)         | Data storage                   |
| **Proxy**    | Nginx                         | Handles frontend + API routing |

GitHub Repository → [three-tier-ecommerce](https://github.com/papani-sivasai/three-tier-ecommerce)

---

## Why Do We Need Nginx?

When both frontend and backend run in Docker, they can talk to each other internally using Docker’s network DNS (e.g., `http://backend:5000`), **but your browser cannot resolve that hostname**.

So, fetching data from `http://backend:5000` in your React app results in:

```
net::ERR_NAME_NOT_RESOLVED
```

Even with exposed ports, browsers only know **localhost**, not container names.

### Nginx Solves This By:

* Serving your React app directly to the browser
* Forwarding `/api` requests to the backend container
* Acting as the **single entry point** ([http://localhost](http://localhost))
* Eliminating **CORS** and DNS issues

Think of **Nginx as the front door** to your entire Dockerized application.

---

## Architecture Diagram

```
Browser
   ↓
Nginx Container  →  Serves React (Frontend)
   ↓
Backend (Express)  →  Connects to MongoDB Atlas
```

Browser only communicates with Nginx
Nginx handles both static assets and API routes
Backend connects securely to MongoDB Atlas

---

## Folder Structure

```bash
three-tier-ecommerce/
│
├── client/          # React frontend
│   ├── Dockerfile
│   ├── .dockerignore
│   └── ...
│
├── server/          # Express backend
│   ├── Dockerfile
│   ├── .dockerignore
│   └── ...
│
├── nginx/           # Nginx configuration
│   └── nginx.conf
│
└── docker-compose.yml
```

---

## Backend (Server) — Dockerfile

```dockerfile
# Use a lightweight Node base image
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy dependency files and install dependencies
COPY package*.json ./
RUN npm install

# Copy the source code
COPY . .

# Expose the server port
EXPOSE 5000

# Start the backend
CMD ["npm", "start"]
```

**Explanation:**

* Uses small `node:18-alpine` image.
* Installs dependencies efficiently.
* App runs on port **5000** inside the container.

---

## Frontend (Client) — Multi-Stage Dockerfile

React apps need to be built and then served. We’ll use a **multi-stage build** for smaller, optimized images.

```dockerfile
# Stage 1: Build the React app
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Serve with Nginx
FROM nginx:alpine
RUN rm -rf /usr/share/nginx/html/*
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**Explanation:**

* Stage 1 → builds the React app.
* Stage 2 → uses Nginx to serve static files.
* Keeps image light and no Node.js in production.

---

## Nginx Configuration

Create a config file at `nginx/nginx.conf`:

```nginx
server {
    listen 80;
    server_name localhost; # Change to your domain if deploying to production

    # Serve React frontend
    location / {
        proxy_pass http://frontend:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_intercept_errors on;
        error_page 404 = /index.html;
        proxy_redirect off;
    }

    # Proxy API requests to backend
    location /api {
        proxy_pass http://backend:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

**Explanation:**

* Serves React files at `/`.
* Forwards `/api` calls to backend.
* Single entry point (`localhost`).

---

## Docker Compose File

Here’s how we connect everything.

```yaml
services:
  backend:
    build: ./server
    container_name: ecommerce-backend
    image: backend
    ports:
      - "5500:5000"
    env_file:
      - ./server/.env
    networks:
      - ecommerce-network
    restart: always

  frontend:
    build: ./client
    container_name: ecommerce-frontend
    image: frontend
    expose:
      - "80"
    env_file:
      - ./client/.env
    depends_on:
      - backend
    networks:
      - ecommerce-network
    restart: always

  nginx:
    image: nginx:alpine
    container_name: ecommerce-nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      - frontend
      - backend
    networks:
      - ecommerce-network

networks:
  ecommerce-network:
    driver: bridge
```

**Explanation:**

* All containers share the same `ecommerce-network`.
* Only **Nginx** exposes port 80 to the host.
* Nginx depends on both frontend and backend.

---

## Environment Files

### Server (`server/.env`)

```ini
# Database Configuration
MONGODB_URI=<Enter your mongo uri here>

# Server Configuration
PORT=5000
NODE_ENV=development

# JWT Configuration
JWT_SECRET=<Enter your jwt here>

# CORS Configuration
CLIENT_URL=http://localhost:5173

# Cloudinary Configuration (for image uploads)
CLOUDINARY_CLOUD_NAME=<Enter your cloud name here>
CLOUDINARY_API_KEY=<Enter your cloud API key here>
CLOUDINARY_API_SECRET=<Enter your cloud API secret here>

# PayPal Configuration
PAYPAL_MODE=sandbox
PAYPAL_CLIENT_ID=<Enter your paypal client id here>
PAYPAL_CLIENT_SECRET=<Enter your paypal client secret here>
```

### Client (`client/.env`)

```ini
CLIENT_URL=http://localhost:5173
VITE_API_BASE_URL=/api
VITE_NODE_ENV=development
```

> Never commit `.env` files publicly. Keep credentials secure.

---

## Run Everything

Build and start all containers:

```bash
docker compose up -d --build
```

Expected output:

```
✔ Network ecommerce-network  Created
✔ Container ecommerce-backend Started
✔ Container ecommerce-frontend Started
✔ Container ecommerce-nginx  Started
```

Now open [http://localhost](http://localhost)
 React app loads
API requests flow through Nginx
Backend connects to MongoDB Atlas
No CORS or DNS issues

---

## Summary

You’ve just containerized a **three-tier MERN app** using everything learned so far:

* Dockerfiles for frontend & backend
* Docker Compose for orchestration
* Shared network communication
* Nginx reverse proxy as single entry point

This setup mirrors production-grade architecture used by modern full-stack apps.

---

With this article, we finished this docker series together. If you have any questions/doubts, please ask me in the comments and I will try to answer every question.

Stay tuned for more upcoming series like this….!!🙂
Thank you.

---

**Author:** *Siva Sai Papani*
**Article Source:** [Containerizing a Three-Tier MERN E-Commerce App on Hashnode](https://docker-part7.hashnode.dev/containerising-a-three-tier-mern-e-commerce-app-with-docker-and-nginx)
