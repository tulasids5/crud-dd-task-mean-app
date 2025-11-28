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

  Docker Hub :  
  Backend → tulasigowda/mean-backend:latest    
  
<img width="1916" height="1018" alt="Screenshot 2025-11-27 233409" src="https://github.com/user-attachments/assets/01b57817-2179-42a1-a0a6-3ca3c05d2ce0" />    
<img width="1919" height="1014" alt="Screenshot 2025-11-28 144443" src="https://github.com/user-attachments/assets/aae1e6e8-eddc-4193-8f13-78da2c69bf63" />

  Frontend → tulasigowda/mean-frontend:latest   
<img width="1919" height="1015" alt="Screenshot 2025-11-27 233628" src="https://github.com/user-attachments/assets/29f61536-f9b5-41de-9884-af6e8002109e" />  
<img width="1914" height="1017" alt="Screenshot 2025-11-28 144429" src="https://github.com/user-attachments/assets/76a504b7-78dc-4fae-9d7b-58666beb57f0" />

  
  ### Docker Compose Deployment on Cloud VM

  An Ubuntu EC2 instance was launched on AWS  
  Docker and Docker Compose installed  
  Repository cloned to server  
  App deployed using:  
  `docker-compose up -d`  
<img width="1919" height="1018" alt="Screenshot 2025-11-28 144114" src="https://github.com/user-attachments/assets/f80c3744-11d0-481b-b461-2d9fa692002e" />

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
<img width="1919" height="1016" alt="Screenshot 2025-11-28 144558" src="https://github.com/user-attachments/assets/49418598-cd4b-4943-8ed0-1c2a148de8e5" />  
<img width="1919" height="1017" alt="Screenshot 2025-11-28 144214" src="https://github.com/user-attachments/assets/afae738b-0701-469b-83b3-436a6d53a6e1" />

## Nginx Reverse Proxy Setup

Nginx configured as reverse proxy inside Docker  
Routes:  
    `/` → Angular frontend  
    `/api` → Node backend  
<img width="1580" height="699" alt="Screenshot 2025-11-28 144833" src="https://github.com/user-attachments/assets/97a6a401-b581-4c07-9423-af4f7a8c9b04" />  
<img width="1544" height="751" alt="Screenshot 2025-11-28 144843" src="https://github.com/user-attachments/assets/d72af929-d86d-4564-bc59-92619ad77243" />

Public access only via:  
`http://<EC2-PUBLIC-IP>/`  
<img width="1910" height="1014" alt="Screenshot 2025-11-27 233818" src="https://github.com/user-attachments/assets/99522c0b-e5ea-4510-93d6-59bf18a4ce18" />  
<img width="1919" height="1013" alt="Screenshot 2025-11-27 233828" src="https://github.com/user-attachments/assets/e8ccd2e0-f6c2-49e4-82f3-c38c0a9aa0e4" />

App fully available on port 80 only
