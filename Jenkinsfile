pipeline {
    agent any
    tools{
        jdk 'Java21'
        maven 'maven3'
    }
    
    environment{
        SCANNER_HOME= tool 'Sonar-Scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'github-pat', url: 'https://github.com/imseshu/Ekart.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn clean compile -DskipTests=true"
            }
        }
        stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Sonarqube') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Devops-Shopping-Cart \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Shopping-Cart '''
                }
            }
        }
        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }
        stage('Docker Build and Push') {
            steps {
                withDockerRegistry(credentialsId: 'Docker', url: 'https://index.docker.io/v1/') {
                    sh "docker build -t shopping-cart -f docker/Dockerfile ."
                    sh "docker tag shopping-cart imseshu12/shopping-cart:latest"
                    sh "docker push imseshu12/shopping-cart:latest"
                }
            }
        }
/*        stage('Docker Deploy') {
            steps {
                withDockerRegistry(credentialsId: 'Docker', url: 'https://index.docker.io/v1/') {
                    sh "docker run -d --name ekart-ekart -p 8070:8070 imseshu12/shopping-cart:latest"
                }
            }
        }*/
    }
}
