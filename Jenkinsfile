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
                    sh '/opt/apache-maven-3.9.9/bin/mvn clean package'
                }
            }
        }

        stage('Execute Pre-query in MySQL') {
            steps {
                script {
                    // SSH into the test server and execute pre-query
                    sh 'ssh jenkins "wget https://github.com/prabisonshakya/inventory/blob/master/resources/postquery.sql && mysql -uroot -proot < prequery.sql"'
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
