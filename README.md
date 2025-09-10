# Chattingo - Real-time Chat Application

A full-stack real-time chat application built with Spring Boot backend and React frontend, featuring WebSocket communication, user authentication, and group messaging capabilities.

## 🏗️ Architecture

- **Frontend**: React 18 with Material-UI, Redux, and WebSocket (STOMP)
- **Backend**: Spring Boot 3.3.1 with WebSocket support
- **Database**: MySQL 8.0
- **Reverse Proxy**: Nginx with SSL termination
- **Containerization**: Docker & Docker Compose
- **CI/CD**: Jenkins Pipeline
- **SSL**: Let's Encrypt certificates via Certbot
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Backend       │    │   Database      │
│   (React)       │◄──►│   (Spring Boot) │◄──►│   (MySQL)       │
│   Port: 80      │    │   Port: 8080    │    │   Port: 3306    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │
         └────── WebSocket ──────┘
```
## 📋 Prerequisites

- Docker & Docker Compose
- Node.js 16+ (for local development)
- Java 17+ (for local development)
- Maven 3.6+ (for local development)

## 🚀 Quick Start with Docker Compose

### 1. Clone the Repository
```bash
git clone https://github.com/DattaRahegaonkar/chattingo-update.git
cd chattingo-update
```

### 2. Environment Setup
```bash
# Copy environment files
cp backend/.env.example backend/.env
cp frontend/.env.example frontend/.env

# Edit backend/.env with your database credentials
# Edit frontend/.env with your API URL
```

### 3. Create Docker Network
```bash
docker network create chattingo-network
```

### 4. Start Services
```bash
docker-compose up -d
```

The application will be available at:
- Frontend: http://localhost (via Nginx)
- Backend API: http://localhost/api
- WebSocket: ws://localhost/ws

## 🛠️ Local Development Setup

### Backend Setup
```bash
cd backend

# Configure database connection in .env
cp .env.example .env

# Install dependencies and run
./mvnw clean install
./mvnw spring-boot:run
```

### Frontend Setup
```bash
cd frontend

# Install dependencies
npm install

# Configure API endpoint
cp .env.example .env

# Start development server
npm start
```

### Database Setup
```bash
# Start MySQL container
docker run -d \
  --name mysql-dev \
  -e MYSQL_ROOT_PASSWORD=yourpassword \
  -e MYSQL_DATABASE=chattingo \
  -p 3306:3306 \
  mysql:latest

# Initialize database with init.sql
mysql -h localhost -u root -p chattingo < backend/init.sql
```

## 🌐 Production Deployment with SSL

### 1. Domain Configuration
Update the following files with your domain:
- `nginx/nginx.conf` - Replace `chattingo.rahegaonkar.site`
- `ssl.sh` - Update domains array and email
- `docker-compose.yml` - Update certbot command

### 2. SSL Certificate Generation
```bash
# Make SSL script executable
chmod +x ssl.sh

# Generate SSL certificates
./ssl.sh
```

### 3. Deploy with SSL
```bash
docker-compose up -d
```

## 🔧 Nginx Reverse Proxy Configuration

The Nginx configuration provides:

- **SSL Termination**: Handles HTTPS certificates
- **Load Balancing**: Routes traffic between services
- **WebSocket Support**: Proxies WebSocket connections
- **Static File Serving**: Serves React build files
- **API Routing**: Routes `/api/*` and `/auth/*` to backend

### Key Routes:
- `/` → React Frontend
- `/api/*` → Spring Boot Backend
- `/auth/*` → Authentication endpoints
- `/ws/*` → WebSocket connections
- `/.well-known/acme-challenge/*` → SSL certificate validation

## 🚀 Jenkins CI/CD Pipeline

```
pipeline {
    agent any
    
    stages {
        stage('Clean workspace') { 
            // Clean workspace fromt the workspace directory
        }         
        stage('Git Clone') { 
            // Clone repository from GitHub
        }
         stage('Creating .env files') { 
            // Environment variables
        }
         stage('OWASP Dependency Check') { 
            // to check the vulernabilities
        }
        stage('Filesystem Scan') { 
            // Security scan of source code 
        }
        stage('Images Building') { 
            // Build Docker images for frontend & backend
        }
        stage('Images Scanning') { 
            // Vulnerability scan of Docker images
        }
        stage('Push to Registry') { 
            // Push images to Docker Hub/Registry 
        }
        stage('Update Compose') { 
            // Update docker-compose with new image tags
        }
        stage('Deploy') { 
            // Deploy to Hostinger VPS
        }
    }
    stage('Post Declaration') {
         // storing the artifacts in jenkins
    }         
}
```

### Pipeline Configuration
```groovy
// Jenkinsfile includes shared library usage
@Library('shared-library') _

// Stages: Clean → Clone → Build → Test → Deploy
```

### Setup Jenkins Pipeline
1. Create Pipeline job
2. Configure GitHub webhook
3. Configured Jenkins Shared Library

## 📊 Monitoring & Health Checks

### Service Health Checks
```bash
# Check all services
docker-compose ps

# Check specific service logs
docker-compose logs -f [service-name]

# Health check endpoints
curl http://localhost/api/health
```

### Database Health Check
```bash
# MySQL health check (configured in docker-compose)
mysqladmin ping -h 127.0.0.1 -u root -p
```

## 🔧 Configuration Files

### Environment Variables

**Backend (.env)**
```env
MYSQL_ROOT_PASSWORD=yourpassword
MYSQL_DATABASE=chattingo
MYSQL_USER=chattingo_user
MYSQL_PASSWORD=userpassword
```

**Frontend (.env)**
```env
REACT_APP_API_URL=https://yourdomain.com
```

### Docker Compose Services
- `dbservice`: MySQL database
- `appservice`: Spring Boot backend
- `web`: React frontend
- `nginx`: Reverse proxy with SSL
- `certbot`: SSL certificate management

## 🐛 Troubleshooting

### Common Issues

**SSL Certificate Issues**
```bash
# Check certificate status
docker-compose logs certbot

# Regenerate certificates
./ssl.sh
```

**Database Connection Issues**
```bash
# Check MySQL logs
docker-compose logs dbservice

# Verify database connectivity
docker-compose exec dbservice mysql -u root -p
```

**WebSocket Connection Issues**
```bash
# Check Nginx WebSocket configuration
docker-compose exec nginx nginx -t

# Verify backend WebSocket endpoint
curl -i -N -H "Connection: Upgrade" -H "Upgrade: websocket" http://localhost/ws
```

## 📝 API Documentation

### Authentication Endpoints
- `POST /auth/login` - User login
- `POST /auth/register` - User registration
- `POST /auth/logout` - User logout

### Chat Endpoints
- `GET /api/chats` - Get user chats
- `POST /api/chats` - Create new chat
- `GET /api/chats/{id}/messages` - Get chat messages

## 🙏 Acknowledgments

- Spring Boot for the robust backend framework
- React for the dynamic frontend
- Material-UI for the component library
- Let's Encrypt for free SSL certificates
- Docker for containerization
