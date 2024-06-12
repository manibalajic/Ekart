pipeline{
    agent any
    options {
        buildDiscarder(logRotator(numToKeepStr: '2', artifactNumToKeepStr: '2'))
        //retry(2)
    }
    tools{
        maven 'maven38'
        jdk 'jdk17'
    }
    environment{
        sonarscan_HOME = tool 'sonarscan'
    }
    
    stages{
        stage("Git checkout"){
        steps{
            git branch: 'main', url: 'https://github.com/manibalajic/Ekart.git'
        }
        }
        stage("compile"){
        steps{
           sh 'mvn compile' 
        }
        }
        stage('test'){
            steps{
                sh 'mvn test -DskipTests=true'
            }
        }
        stage('sonarscan'){
        steps{
         withSonarQubeEnv('sonarserver') {
             sh ''' $sonarscan_HOME/bin/sonar-scanner \
                         -Dsonar.projectName=eKart \
                        -Dsonar.projectKey=eKart \
                        -Dsonar.java.binaries=. '''

            }
          }
        }
        
        stage('OWASP'){
            steps{
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'dc'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
        }
        stage("build"){
            steps{
                sh 'mvn package -DskipTests=true'
            }
        }
        stage('deploy'){
        steps{
            withMaven(globalMavenSettingsConfig: 'globalmaven', jdk: 'jdk17', maven: 'maven38', mavenSettingsConfig: '', traceability: true) {
                sh 'mvn deploy -DskipTests=true'
                }
        }
        }
        stage("docker-build"){
            steps{
                withDockerRegistry(credentialsId: 'docker-cred', url:'') {
                sh 'docker build -t dockerluv/manibalajic:ekart -f docker/Dockerfile . '
                  
                }
            }
        }
        stage("trivy"){
        steps{
            sh 'trivy image dockerluv/manibalajic:ekart > trivy_report.txt'
        }
        }
        stage("docker-push"){
            steps{
                withDockerRegistry(credentialsId: 'docker-cred', url:'') {
                sh 'docker push dockerluv/manibalajic:ekart '    
                }
            }
        }
        stage("kube-deploy"){
            steps{
                sh 'kubectl create -f deploymentservice.yml'
                sh 'kubectl get service '
                    
                }
            }
        }
        
    }
