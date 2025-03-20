# Restify Docker Setup Guide

This guide will walk you through setting up a Restify server inside a Docker container, attaching it to a custom network, and running it.

---

## **1. Prerequisites**
Make sure you have the following installed:
- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/) (optional)
- Node.js & npm (if running locally before containerizing)
- Restify installed
```
npn init -y
npm install restify
```
---

## **2. Project Structure**
Ensure your project directory contains the following files:
```
restify-app/
│── node_modules/      # (generated after npm install)
│── package.json
│── package-lock.json
│── server.js
│── Dockerfile
```

### **server.js (Example Restify Server)**
Ensure your `server.js` listens on **0.0.0.0** to allow access from Docker:
```javascript
const restify = require('restify');
const server = restify.createServer();

server.use(restify.plugins.bodyParser());

server.get('/', (req, res, next) => {
    res.send({ message: 'Restify server is running!' });
    next();
});

server.post('/data', (req, res, next) => {
    res.send({ message: 'Data received!', data: req.body });
    next();
});

server.listen(3000, '0.0.0.0', () => {
    console.log('Restify server running on port 3000');
});
```

---



## **3. Create a Dockerfile**
Create a `Dockerfile` in the same directory:
```dockerfile
# Use official Node.js image
FROM node:18-alpine

# Set working directory in the container
WORKDIR /app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application files
COPY . .

# Expose the required port
EXPOSE 3000

# Start the Restify server
CMD ["node", "server.js"]
```

---

## **4. Create and Attach to a Docker Network**

### **Attach to the MES Docker Network**
Run the following commands to get the MES_NETWORK name:
```sh
docker network ls
```


### **Build and Run the Restify Container**
Run the following commands to build and start your container:
```sh
docker build -t restify-app .
docker run -d --name restify-container --network MES_NETWORK -p 3000:3000 restify-app
```

---


## **5. Verify the Setup**
### **Check Running Containers**
```sh
docker ps
```

### **Test the API from the Host**
Use `curl` to test the API:
```sh
curl http://localhost:3000/
```
Expected response:
```json
{ "message": "Restify server is running!" }
```


### **Test from Another Docker Container in the Same Network**
Start a temporary container inside `my_network` and test the connection:
```sh
docker run --rm --network my_network curlimages/curl curl http://restify-container:3000/
```


---

## **6. Updating the Restify Server**
If you modify `server.js`, you need to rebuild and restart the container:
```sh
docker stop restify-container
docker rm restify-container
docker build -t restify-app .
docker run -d --name restify-container --network my_network -p 3000:3000 restify-app
```



### **Using Volume Mounting for Easier Development**
Instead of rebuilding every time, mount the code as a volume:
```sh
docker run -d --name restify-container --network my_network -p 3000:3000 -v "$(pwd):/app" restify-app
```
This way, changes to `server.js` will reflect when you restart the container:
```sh
docker restart restify-container
```


---



## **7. Stopping and Removing the Container**
To stop the container:
```sh
docker stop restify-container
```
To remove the container:
```sh
docker rm restify-container
```
To remove the network (if needed):
```sh
docker network rm my_network
```

---

## **8. Cleanup**
To remove all unused Docker images, containers, and networks:
```sh
docker system prune -a
```

---

