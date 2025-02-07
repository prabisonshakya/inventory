pipeline {
    agent any

    stages {
        stage('SCM') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    // Perform SonarQube analysis using Maven
                    def mvn = tool 'Default Maven';
                    withSonarQubeEnv() {
                        sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=jenkins"
                    }
                }
            }
        }

        stage('Build and Package') {
            steps {
                script {
                    // Maven clean install
                    sh 'mvn clean package'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Change to the Docker directory and build the Docker image
                    sh 'cd /opt/project/code-with-quarkus/src/main/docker/ && docker build --no-cache --rm --squash -t 192.168.3.91:5000/inventory:v1.0.0 .'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Push the Docker image to the registry
                    sh 'docker push 10.120.2.228:5000/tomcat:v1'
                }
            }
        }

        stage('Pull Docker Image on Test Server') {
            steps {
                script {
                    // SSH into the test server and pull the Docker image
                    sh 'ssh jenkins "docker pull 10.120.2.228:5000/tomcat:v1"'
                }
            }
        }

        stage('Execute Pre-query in MySQL') {
            steps {
                script {
                    // SSH into the test server and execute pre-query
                    sh 'ssh jenkins "wget https://raw.githubusercontent.com/prabisonshakya/docker-jenkins/main/resources/prequery.sql && mysql -uroot -proot < prequery.sql"'
                }
            }
        }

        stage('Run Docker Compose') {
            steps {
                script {
                    // SSH into the test server and run Docker Compose
                    sh 'ssh jenkins "wget https://raw.githubusercontent.com/prabisonshakya/docker-jenkins/main/src/main/docker/docker-compose.yml && docker-compose up -d"'
                }
            }
        }

        stage('Execute Post-query in MySQL') {
            steps {
                script {
                    // SSH into the test server and execute post-query
                    sh 'ssh jenkins "wget https://raw.githubusercontent.com/prabisonshakya/docker-jenkins/main/resources/postquery.sql && mysql -uroot -proot < postquery.sql"'
                }
            }
        }
    }
}
