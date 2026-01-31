pipeline {
    agent any
    environment {
        SONARQUBE_SERVER = 'http://localhost:9000'
        DOCKER_IMAGE = 'your-docker-image-name' 
    }
    stages {
        stage("Git Checkout"){
            steps{
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/girish-sakore/spring-petclinic.git'
            }
        }

        stage('Compile with Maven') {
            steps {
                echo "======== Compiling with Maven ========"
                sh 'mvn clean compile'
            }
        }
        stage('Run Tests') {
            steps {
                echo "======== Running Tests ========"
                sh 'mvn test'
            }
        }
        stage('SonarQube Scan') {
            steps {
                echo "======== Running SonarQube Scan ========"
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        stage('Build with Maven') {
            steps {
                echo "======== Building with Maven ========"
                sh 'mvn package'
            }
        }
        stage('Docker Image Creation') {
            steps {
                echo "======== Creating Docker Image ========"
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }
        stage('Trivy Scan') {
            steps {
                echo "======== Running Trivy Scan ========"
                sh 'trivy image $DOCKER_IMAGE'
            }
        }
        stage('Deploy') {
            steps {
                echo "======== Deploying Application ========"
                // Add your deployment commands here (e.g., kubectl, docker-compose, etc.)
                sh 'kubectl apply -f deployment.yaml' // Example for Kubernetes
            }
        }
    }
    post {
        always {
            echo "======== Pipeline Completed ========"
        }
        success {
            echo "======== Pipeline Executed Successfully ========"
        }
        failure {
            echo "======== Pipeline Execution Failed ========"
        }
    }
}