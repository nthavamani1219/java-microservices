pipeline {
     agent
    {
        label 'Linux_Slave-new'
    }

    environment {
        DOCKER_REGISTRY = 'your-docker-registry'
        JAVA_HOME = '/usr/lib/jvm/openlogic-openjdk-21-hotspot'
        MAVEN_HOME = '/data/admin/jenkins/.jenkins/tools/hudson.tasks.Maven_MavenInstallation/Maven_3.9.2'
        KUBE_CONFIG = credentials('kubeconfig')
        DOCKER_USER = thavamani0212
        DOCKER_PASS = Thavamani126@
        SERVICE_NAME = 'stockmanagerservice'
        SERVICE_PORT = '8030'
        SONAR = 'Sonar'
        SCANNER_HOME = tool 'SonarQube Scanner 4.7';
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build and Test') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_REGISTRY}/${SERVICE_NAME}:${BUILD_NUMBER} ."
            }
        }
        stage('Scan Docker Image') {
            steps {
                sh '''
                    echo "Scanning image for vulnerabilities..."
                    trivy image --severity CRITICAL,HIGH ${DOCKER_REGISTRY}/${SERVICE_NAME}:${BUILD_NUMBER}
                '''
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-registry-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "docker login -u $DOCKER_USER -p $DOCKER_PASS ${DOCKER_REGISTRY}"
                    sh "docker push ${DOCKER_REGISTRY}/${SERVICE_NAME}:${BUILD_NUMBER}"
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    dir('../kubernetes') {
                        sh "kubectl apply -f ${SERVICE_NAME}-service.yaml"
                        sh "kubectl set image deployment/${SERVICE_NAME} ${SERVICE_NAME}=${DOCKER_REGISTRY}/${SERVICE_NAME}:${BUILD_NUMBER}"
                    }
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh "kubectl rollout status deployment/${SERVICE_NAME}"
                    sh "curl -s http://localhost:${SERVICE_PORT}/health"
                }
            }
        }
    }
    
    post {
        success {
            mail to: 'n.thavamani@nagarro.com',
                subject: "SUCCESS | Deployment Completed for ${JOB_BASE_NAME}",
                body: "Hi,\n\n\
                The deployment has been completed for the Jenkins pipeline '${JOB_BASE_NAME}'.\n\n\
                The Application URL and Pipeline details are below.\n\n\
                Status: SUCCESS\n\
                Pipeline: ${JOB_NAME}\n\
                Build No: ${BUILD_NUMBER}\n\n"
        }
        failure {
            mail to: 'n.thavamani@nagarro.com',
            subject: "Failed | Deployment  Failed for ${JOB_BASE_NAME}",
            body: "Hi,\n\n\n
            The deployment has failed for the Jenkins pipeline '${JOB_BASE_NAME}'. Please check, Below are the details.\n\n\n
            Status: FAILED\n\n
            Pipeline: ${JOB_NAME}\n\n
            Build No: ${BUILD_NUMBER}\n\n
            Build URL: ${BUILD_URL}\n\n\n"
        }
    }
} 
