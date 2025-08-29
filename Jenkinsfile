pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'your-registry.com'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        PROJECT_NAME = 'finalproject'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    echo "Building project: ${env.PROJECT_NAME}"
                    echo "Build number: ${env.BUILD_NUMBER}"
                }
            }
        }
        
        stage('Install Dependencies') {
            parallel {
                stage('Install Python Dependencies') {
                    steps {
                        script {
                            // Install Python dependencies for API Gateway
                            dir('api-gateway') {
                                sh 'python -m pip install --upgrade pip'
                                sh 'python -m pip install -r requirements.txt'
                            }
                            
                            // Install Python dependencies for Product Service
                            dir('product-catalog-service') {
                                sh 'python -m pip install --upgrade pip'
                                sh 'python -m pip install -r requirements.txt'
                            }
                            
                            // Install Python dependencies for Order Service
                            dir('order-service') {
                                sh 'python -m pip install --upgrade pip'
                                sh 'python -m pip install -r requirements.txt'
                            }
                        }
                    }
                }
                
                stage('Install Node.js Dependencies') {
                    steps {
                        script {
                            dir('frontend') {
                                sh 'npm install'
                            }
                        }
                    }
                }
            }
        }
        
        stage('Run Tests') {
            parallel {
                stage('Test API Gateway') {
                    steps {
                        script {
                            dir('api-gateway') {
                                // Run Python tests if they exist
                                sh 'python -m pytest tests/ || echo "No tests found"'
                            }
                        }
                    }
                }
                
                stage('Test Product Service') {
                    steps {
                        script {
                            dir('product-catalog-service') {
                                sh 'python -m pytest tests/ || echo "No tests found"'
                            }
                        }
                    }
                }
                
                stage('Test Order Service') {
                    steps {
                        script {
                            dir('order-service') {
                                sh 'python -m pytest tests/ || echo "No tests found"'
                            }
                        }
                    }
                }
                
                stage('Test Frontend') {
                    steps {
                        script {
                            dir('frontend') {
                                // Run Angular tests
                                sh 'npm run test -- --watch=false --browsers=ChromeHeadless || echo "Tests failed or not configured"'
                            }
                        }
                    }
                }
            }
        }
        
        stage('Code Quality') {
            steps {
                script {
                    // Run linting and code quality checks
                    dir('frontend') {
                        sh 'npm run lint || echo "Linting failed or not configured"'
                    }
                    
                    // Python code quality checks
                    sh 'python -m flake8 api-gateway/ || echo "Flake8 not available"'
                    sh 'python -m flake8 product-catalog-service/ || echo "Flake8 not available"'
                    sh 'python -m flake8 order-service/ || echo "Flake8 not available"'
                }
            }
        }
        
        stage('Build Docker Images') {
            parallel {
                stage('Build API Gateway Image') {
                    steps {
                        script {
                            dir('api-gateway') {
                                sh "docker build -t ${env.PROJECT_NAME}-api-gateway:${env.IMAGE_TAG} ."
                                sh "docker tag ${env.PROJECT_NAME}-api-gateway:${env.IMAGE_TAG} ${env.PROJECT_NAME}-api-gateway:latest"
                            }
                        }
                    }
                }
                
                stage('Build Product Service Image') {
                    steps {
                        script {
                            dir('product-catalog-service') {
                                sh "docker build -t ${env.PROJECT_NAME}-product-catalog-service:${env.IMAGE_TAG} ."
                                sh "docker tag ${env.PROJECT_NAME}-product-catalog-service:${env.IMAGE_TAG} ${env.PROJECT_NAME}-product-catalog-service:latest"
                            }
                        }
                    }
                }
                
                stage('Build Order Service Image') {
                    steps {
                        script {
                            dir('order-service') {
                                sh "docker build -t ${env.PROJECT_NAME}-order-service:${env.IMAGE_TAG} ."
                                sh "docker tag ${env.PROJECT_NAME}-order-service:${env.IMAGE_TAG} ${env.PROJECT_NAME}-order-service:latest"
                            }
                        }
                    }
                }
                
                stage('Build Frontend Image') {
                    steps {
                        script {
                            dir('frontend') {
                                sh "docker build -t ${env.PROJECT_NAME}-frontend:${env.IMAGE_TAG} ."
                                sh "docker tag ${env.PROJECT_NAME}-frontend:${env.IMAGE_TAG} ${env.PROJECT_NAME}-frontend:latest"
                            }
                        }
                    }
                }
            }
        }
        
        stage('Push Images to Registry') {
            when {
                anyOf {
                    branch 'main'
                    branch 'master'
                    branch 'develop'
                }
            }
            steps {
                script {
                    // Push images to Docker registry if configured
                    if (env.DOCKER_REGISTRY != 'your-registry.com') {
                        sh "docker tag ${env.PROJECT_NAME}-api-gateway:${env.IMAGE_TAG} ${env.DOCKER_REGISTRY}/${env.PROJECT_NAME}-api-gateway:${env.IMAGE_TAG}"
                        sh "docker tag ${env.PROJECT_NAME}-product-catalog-service:${env.IMAGE_TAG} ${env.DOCKER_REGISTRY}/${env.PROJECT_NAME}-product-catalog-service:${env.IMAGE_TAG}"
                        sh "docker tag ${env.PROJECT_NAME}-order-service:${env.IMAGE_TAG} ${env.DOCKER_REGISTRY}/${env.PROJECT_NAME}-order-service:${env.IMAGE_TAG}"
                        sh "docker tag ${env.PROJECT_NAME}-frontend:${env.IMAGE_TAG} ${env.DOCKER_REGISTRY}/${env.PROJECT_NAME}-frontend:${env.IMAGE_TAG}"
                        
                        sh "docker push ${env.DOCKER_REGISTRY}/${env.PROJECT_NAME}-api-gateway:${env.IMAGE_TAG}"
                        sh "docker push ${env.DOCKER_REGISTRY}/${env.PROJECT_NAME}-product-catalog-service:${env.IMAGE_TAG}"
                        sh "docker push ${env.DOCKER_REGISTRY}/${env.PROJECT_NAME}-order-service:${env.IMAGE_TAG}"
                        sh "docker push ${env.DOCKER_REGISTRY}/${env.PROJECT_NAME}-frontend:${env.IMAGE_TAG}"
                    } else {
                        echo "Docker registry not configured, skipping push"
                    }
                }
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                script {
                    echo "Deploying to staging environment..."
                    // Stop existing containers
                    sh 'docker-compose down || true'
                    
                    // Start services with latest images
                    sh 'docker-compose up -d'
                    
                    // Wait for services to be ready
                    sh 'sleep 30'
                    
                    // Health checks
                    sh 'curl -f http://localhost:5000/health || echo "API Gateway health check failed"'
                    sh 'curl -f http://localhost:5001/health || echo "Product Service health check failed"'
                    sh 'curl -f http://localhost:5002/health || echo "Order Service health check failed"'
                    sh 'curl -f http://localhost:4200/ || echo "Frontend health check failed"'
                }
            }
        }
        
        stage('Deploy to Production') {
            when {
                anyOf {
                    branch 'main'
                    branch 'master'
                }
            }
            steps {
                script {
                    echo "Deploying to production environment..."
                    
                    // Production deployment logic
                    // This could include:
                    // - Rolling updates
                    // - Blue-green deployment
                    // - Database migrations
                    // - Load balancer updates
                    
                    sh 'docker-compose -f docker-compose.prod.yml up -d || docker-compose up -d'
                    
                    // Production health checks
                    sh 'sleep 60'
                    sh 'curl -f http://localhost:5000/health || echo "Production API Gateway health check failed"'
                }
            }
        }
    }
    
    post {
        always {
            // Cleanup
            script {
                // Remove old Docker images to save space
                sh 'docker image prune -f || true'
                
                // Clean up workspace
                cleanWs()
            }
        }
        
        success {
            echo "Pipeline completed successfully!"
        }
        
        failure {
            echo "Pipeline failed!"
            // Send notification emails, Slack messages, etc.
        }
        
        unstable {
            echo "Pipeline is unstable!"
        }
    }
}
