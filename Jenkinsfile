pipeline {
    agent any
    environment {
        SONARQUBE_SERVER = 'http://localhost:9000'
        DOCKER_IMAGE = 'petclinic-image:latest' 
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
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                        timeout(time: 5, unit: 'MINUTES') {
                            sh 'mvn test'
                        }
                    }
                }
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName= petshop \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey= petshop '''
                }
            }
        }
        stage("quality gate"){
            steps {
                script {
                  waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
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