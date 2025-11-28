In this DevOps task, you need to build and deploy a full-stack CRUD application using the MEAN stack (MongoDB, Express, Angular 15, and Node.js). The backend will be developed with Node.js and Express to provide REST APIs, connecting to a MongoDB database. The frontend will be an Angular application utilizing HTTPClient for communication.  

The application will manage a collection of tutorials, where each tutorial includes an ID, title, description, and published status. Users will be able to create, retrieve, update, and delete tutorials. Additionally, a search box will allow users to find tutorials by title.

## Project setup

### Node.js Server

cd backend

npm install

You can update the MongoDB credentials by modifying the `db.config.js` file located in `app/config/`.

Run `node server.js`

### Angular Client

cd frontend

npm install

Run `ng serve --port 8081`

You can modify the `src/app/services/tutorial.service.ts` file to adjust how the frontend interacts with the backend.

Navigate to `http://localhost:8081/`


## Assignment Documentation — MEAN CRUD App Deployment

## Repository Setup

  A new GitHub repository was created for this project  
  Full MEAN codebase (frontend + backend) was pushed  

  GitHub Repository:
  https://github.com/tulasids5/crud-dd-task-mean-app

## Containerization & Deployment
  ### Dockerization Completed

  A Dockerfile was created for backend (Node/Express)  
  A Dockerfile was created for frontend (Angular)  

  These images were built and pushed to Docker Hub:

  Docker Hub Links:  
  Backend → tulasigowda/mean-backend:latest  
  Frontend → tulasigowda/mean-frontend:latest

  ### Docker Compose Deployment on Cloud VM

  An Ubuntu EC2 instance was launched on AWS  
  Docker and Docker Compose installed  
  Repository cloned to server  
  App deployed using:  
  `docker-compose up -d`

All services run automatically (frontend + backend + mongodb + nginx)

## Database Setup

MongoDB was deployed using the official MongoDB Docker image within Docker Compose.

Backend connects via:
`mongodb://mongo:27017/cruddb`

Ensures portability and easy automation

## CI/CD Pipeline Configuration

Jenkins Pipeline implemented for full automation  
Trigger: Code push to GitHub

Pipeline Steps:  
    Pull latest code from GitHub  
    Build Docker images (frontend + backend)  
    Push updated images to Docker Hub  
    SSH into VM  
    Pull latest images & restart containers

Completely automated deployment — Zero manual steps after code push

`pipeline {  
    agent any

    environment {
        DOCKERHUB = credentials('dockerhub')
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/tulasids5/crud-dd-task-mean-app.git'
            }
        }

        stage('Build Backend Image') {
            steps {
                sh 'docker build -t $DOCKERHUB_USR/mean-backend:latest ./backend'
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh 'docker build -t $DOCKERHUB_USR/mean-frontend:latest ./frontend'
            }
        }

        stage('Login to Docker Hub & Push Images') {
            steps {
                sh 'echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin'
                sh 'docker push $DOCKERHUB_USR/mean-backend:latest'
                sh 'docker push $DOCKERHUB_USR/mean-frontend:latest'
            }
        }

        stage('Deploy on VM') {
            steps {
                sshagent(['ssh-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@34.224.174.132 '
                    cd crud-dd-task-mean-app &&
                    docker-compose down &&
                    docker system prune -f &&
                    git pull &&
                    docker-compose pull &&
                    docker-compose up -d
                    '
                    """
                }
            }
        }
    }
}

## Nginx Reverse Proxy Setup

Nginx configured as reverse proxy inside Docker  
Routes:  
    `/` → Angular frontend  
    `/api` → Node backend

Public access only via:  
`http://<EC2-PUBLIC-IP>/`

App fully available on port 80 only
