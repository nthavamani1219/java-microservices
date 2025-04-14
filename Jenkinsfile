pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'your-docker-registry'
        KUBE_CONFIG = credentials('kubeconfig')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build and Test') {
            parallel {
                stage('StockManager') {
                    steps {
                        dir('stockmanager') {
                            sh 'mvn clean package'
                        }
                    }
                }
                stage('ShopFront') {
                    steps {
                        dir('shopfront') {
                            sh 'mvn clean package'
                        }
                    }
                }
                stage('ProductCatalogue') {
                    steps {
                        dir('productcatalogue') {
                            sh 'mvn clean package'
                        }
                    }
                }
            }
        }
        
        stage('Build Docker Images') {
            parallel {
                stage('StockManager Image') {
                    steps {
                        dir('stockmanager') {
                            sh 'docker build -t ${DOCKER_REGISTRY}/stockmanager:${BUILD_NUMBER} .'
                        }
                    }
                }
                stage('ShopFront Image') {
                    steps {
                        dir('shopfront') {
                            sh 'docker build -t ${DOCKER_REGISTRY}/shopfront:${BUILD_NUMBER} .'
                        }
                    }
                }
                stage('ProductCatalogue Image') {
                    steps {
                        dir('productcatalogue') {
                            sh 'docker build -t ${DOCKER_REGISTRY}/productcatalogue:${BUILD_NUMBER} .'
                        }
                    }
                }
            }
        }
        
        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-registry-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS ${DOCKER_REGISTRY}'
                    sh 'docker push ${DOCKER_REGISTRY}/stockmanager:${BUILD_NUMBER}'
                    sh 'docker push ${DOCKER_REGISTRY}/shopfront:${BUILD_NUMBER}'
                    sh 'docker push ${DOCKER_REGISTRY}/productcatalogue:${BUILD_NUMBER}'
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    dir('kubernetes') {
                        sh 'kubectl apply -f stockmanager-service.yaml'
                        sh 'kubectl apply -f shopfront-service.yaml'
                        sh 'kubectl apply -f productcatalogue-service.yaml'
                        
                        // Update image versions in deployments
                        sh 'kubectl set image deployment/stockmanager stockmanager=${DOCKER_REGISTRY}/stockmanager:${BUILD_NUMBER}'
                        sh 'kubectl set image deployment/shopfront shopfront=${DOCKER_REGISTRY}/shopfront:${BUILD_NUMBER}'
                        sh 'kubectl set image deployment/productcatalogue productcatalogue=${DOCKER_REGISTRY}/productcatalogue:${BUILD_NUMBER}'
                    }
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh 'kubectl rollout status deployment/stockmanager'
                    sh 'kubectl rollout status deployment/shopfront'
                    sh 'kubectl rollout status deployment/productcatalogue'
                }
            }
        }
    }
    
    post {
        always {
            // Clean up workspace
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
} 