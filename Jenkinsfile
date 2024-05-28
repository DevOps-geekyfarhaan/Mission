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
        stage('Trivy Scan FS') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Mission -Dsonar.projectName=Mission -Dsonar.java.binaries=. 
                    '''
                }
            }
        }
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        stage('Deploy Artifacts to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-setting', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t geekyfarhaan/mission:latest ."
                    }
                }
            }
        }
        stage('Trivy Scan Image') {
            steps {
                sh "trivy fs --format table -o trivy-image-report.html geekyfarhaan/mission:latest"
            }
        }
        stage('Publish Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push geekyfarhaan/mission:latest"
                    }
                }
            }
        }
        stage('Deploy to k8') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'DS-EKS', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://2A304992B389245D72BBDAC45645FC6F.gr7.ap-south-1.eks.amazonaws.com') {
                    sh "kubectl apply -f ds.yml -n webapps"
                    sleep 60
                }
            }
        }
        stage('Verigy Deploymnet') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'DS-EKS', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://2A304992B389245D72BBDAC45645FC6F.gr7.ap-south-1.eks.amazonaws.com') {
                    sh "kubectl get pods -n webapps"
                    sh "kubectl get svc -n webapps"
                }
            }
        }
    }
    post {
    always {
        script {
            def jobName = env.JOB_NAME
            def buildNumber = env.BUILD_NUMBER
            def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
            def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'
            def body = """
                <html>
                <body>
                <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                <h2>${jobName} - Build ${buildNumber}</h2>
                <div style="background-color: ${bannerColor}; padding: 10px;">
                <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                </div>
                <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                </div>
                </body>
                </html>
            """
            emailext (
                subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                body: body,
                to: 'farhaankhan.jio@gmail.com',
                from: 'farhaankhan.jio@gmail.com',
                replyTo: 'jenkins@example.com',
                mimeType: 'text/html',
                attachmentsPattern: 'trivy-image-report.html'
            )
        }
    }
}
}
