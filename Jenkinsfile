pipeline {
    agent any
    
    tools {
        jdk 'jdk11'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/sagarkulkarni1989/Petclinic.git'
            }
        }
        stage('Code Compile') {
            steps {
                sh "mvn clean compile"
            }
        }
        stage('Unit test and jacoco') {
            steps {
                sh "mvn test"
            }
            post {
              always {
                    junit 'target/surefire-reports/*.xml'
                    jacoco execPattern: 'target/jacoco.exec'
              }
            }
        }
        stage('sonarqube Analysis') {
            steps {
              withSonarQubeEnv('sonar-server') {
                 sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petclinic \
                 -Dsonar.java.binaries=. \
                 -Dsonar.projectKey=Petclinic '''
                 
                }
            }
           
        }
        // stage("OWASP Dependency Check"){
        //     steps{
        //         dependencyCheck additionalArguments: '--scan ./ --format HTML ', odcInstallation: 'DP-Check'
        //         dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        //     }
        // }
        stage("Build"){
            steps{
                sh " mvn clean install"
            }
        }
        stage("Docker Build"){
            steps{
                script {
                    
                    withDockerRegistry(credentialsId: 'docker-creds', toolName: 'docker') {
                        
                        sh "docker build -t petclinic ."
                        sh "docker tag petclinic gita/petclinic:latest "
                        sh "docker push gita/petclinic:latest "
   
                    }
                }
            }
        }
        stage('Deploy using Docker') {
            steps {
                sh "docker run -d --name pet1 -p 8082:8082 gita/petclinic:latest"
            }
        }
    }
}
