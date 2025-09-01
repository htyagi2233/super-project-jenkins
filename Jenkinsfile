pipeline{
    agent any
    environment{
        cred = credentials('k8s-133')
        DOCKER_IMAGE = 'htyagi2233/superproject-jenkins'
    }
    options {
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '30', numToKeepStr: '5')
        timeout(30)
    }
    tools {
        maven 'Maven'
    }
    stages{
        stage('checkout stage'){
            steps{
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/htyagi2233/super-project-jenkins.git']])
            }
        }
        stage('SonarQube Analysis'){
            steps{
                script{
                    def mvn = tool 'Maven';
                    withSonarQubeEnv(installationName: 'sonarqube-server') {
                      sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=super-project -Dsonar.projectName='super-project'"
                    }
                }
            }
        }
        stage('Maven build'){
            steps{
                sh 'mvn package'
            }
        }
        stage('Nexus Test'){
            steps{
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: '192.168.192.136:8081',
                    groupId: 'addressbook',
                    version: '2.0-SNAPSHOT',
                    repository: 'maven-snapshots',
                    credentialsId: 'nexus-cred',
                    artifacts: [
                        [artifactId: 'super-project',
                         classifier: '',
                         file: 'target/addressbook-2.0.war',
                         type: 'war']
                    ]
                )
            }
        }
        stage('docker build'){
            steps{
                sh "docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} ."
            }
        }
        stage('DockerHub Image Push'){
            withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USERNAME')]) {
                sh "docker login -u ${DOCKERHUB_USERNAME} -p ${DOCKERHUB_PASSWORD}"
                sh "docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}"
                sh "docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest"
                sh "docker push ${DOCKER_IMAGE}:latest"
            }
        }
    }
}
