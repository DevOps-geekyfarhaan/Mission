pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, credentialsId: 'git-cred', poll: false, url: 'https://github.com/DevOps-geekyfarhaan/Mission.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Test') {
            steps {
                sh "mvn test -DskipTests=True"
            }
        }
        stage('Trivy Scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        // stage('SonarQube Analysis') {
        //     steps {
        //         echo 'Hello World'
        //     }
        // }
        // stage('Hello') {
        //     steps {
        //         echo 'Hello World'
        //     }
        // }
        // stage('Hello') {
        //     steps {
        //         echo 'Hello World'
        //     }
        // }
        // stage('Hello') {
        //     steps {
        //         echo 'Hello World'
        //     }
        // }
    }
}
