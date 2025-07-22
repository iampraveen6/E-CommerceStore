# Project Instructions for E-CommerceStore


## Architecture & Service Boundaries
- **Microservices MERN stack:**
  - `backend/user-service` (users/auth, port 3001)
  - `backend/product-service` (products/categories, port 3002)
  - `backend/cart-service` (cart, port 3003)
  - `backend/order-service` (orders/payments, port 3004)
  - `frontend` (React, port 3000)
- **Communication:** RESTful HTTP APIs, URLs from `.env` files. No direct DB access across services.
- **Data flow:** User → Product → Cart → Order; order placement triggers cross-service validation and stock updates.

## Developer Workflows
- **Install:** `npm install` in root and each service dir
- **Run all:** `npm run dev` (root, if script exists)
- **Run individually:** `cd backend/<service> && npm start` or `cd frontend && npm start`
- **Env setup:** Each service and frontend needs its own `.env` (see `README.md` for examples)
- **API testing:** Use Postman/curl; see `README.md` for endpoint and payload examples

## Project Conventions & Patterns
- **Backend:**
  - Each service is self-contained (models, routes, server.js)
  - MongoDB via Mongoose
  - JWT auth in user-service; other services trust user-service tokens
  - Cross-service calls: Axios, URLs from env
  - Consistent JSON error responses
- **Frontend:**
  - React 18, React Router, Context API, React Query
  - API calls via `src/services/api.js` using env URLs
  - State: Context for auth/cart, React Query for data
  - Responsive CSS in `src/index.css`; grid/card UI patterns
  - Loading and error states are explicit in UI

## Integration & Extensibility
- **Service URLs:** Never hardcode; always use env variables
- **Adding endpoints/services:** Update `.env` and `frontend/src/services/api.js`
- **Health checks:** `/health` endpoint on each service
- **Cross-service logic:** See `order-service/routes/orders.js` for order placement/validation

## References
- **API endpoints:** See `README.md` and each service's `routes/`
- **Frontend API usage:** `frontend/src/services/api.js`, `pages/`
- **Project structure:** `README.md` (Project Structure section)
- **Env config:** `.env` examples in `README.md` and `frontend/.env`

---
If any workflow or convention is unclear, check the referenced files or ask for clarification.

## Containerization & Kubernetes/EKS

### Dockerfile & Containerization
1. **Each service and the frontend has its own `dockerfile`** (see `backend/*/dockerfile`, `frontend/dockerfile`).
2. **Dockerfile pattern:**
   - Use `node:16-alpine` as the base image.
   - Set `WORKDIR /app`.
   - Copy `package*.json` and run `npm install --production`.
   - Copy the rest of the source code.
   - Expose the correct port (e.g., `EXPOSE 3001` for user-service).
   - Set the default command: `CMD ["npm", "start"]`.
   - Example:
     ```dockerfile
     FROM node:16-alpine
     WORKDIR /app
     COPY package*.json ./
     RUN npm install --production
     COPY . .
     EXPOSE 3001
     CMD ["npm", "start"]
     ```
3. **Build and run containers locally:**
   - Build: `docker build -t user-service:latest .`
   - Run: `docker run -p 3001:3001 --env-file .env user-service:latest`

### Kubernetes (k8s)
1. **Manifests are in `k8s/`** (e.g., `user-service.yaml`, `mongo.yaml`, `frontend.yaml`).
2. **Each service has its own Deployment and Service manifest.**
3. **Environment variables** for service URLs and DB connections are set in the manifests (or via ConfigMaps/Secrets).
4. **MongoDB** runs as a separate pod/service (`mongo.yaml`).
5. **Apply all manifests:**
   - `kubectl apply -f k8s/`
6. **Check pods and services:**
   - `kubectl get pods`
   - `kubectl get services`

### EKS (AWS Kubernetes)
1. **Push images to a registry** accessible by EKS (e.g., Amazon ECR):
   - Tag and push each service image after building.
2. **Update image references** in k8s manifests to use the ECR image URLs.
3. **Deploy using standard Kubernetes manifests:**
   - `kubectl apply -f k8s/`
4. **Configure secrets and environment variables** using Kubernetes Secrets/ConfigMaps.
5. **Set up load balancers and persistent storage** as needed for production.

### References & Examples
- See `README.md` (Deployment, Docker, and k8s sections) and the `k8s/` directory for concrete examples.
- Example Dockerfile and manifest snippets are provided in the README.
