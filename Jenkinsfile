pipeline {
    agent any
    environment {
        SONARQUBE_SERVER = 'http://localhost:9000'
        SONARQUBE_CREDENTIALS = 'squ_1ba96ddad3ab5a739ecb747283bd9b33fbca5eae'
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
                // script {
                //     catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                //         timeout(time: 5, unit: 'MINUTES') {
                //             sh 'mvn test'
                //         }
                //     }
                // }
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                echo "======== Sonarqube Analysis ========"
                withSonarQubeEnv('sonar-server') {
                    sh ''' mvn clean verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
                        -Dsonar.projectKey=petshop \
                        -Dsonar.projectName='petshop' \
                        -Dsonar.host.url=$SONARQUBE_SERVER \
                        -Dsonar.token=$SONARQUBE_CREDENTIALS'''
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