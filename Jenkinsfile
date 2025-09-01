pipeline{
    agent any
    environment{
        cred = credentials('k8s-133')
        KUBECONFIG = "${WORKSPACE}/kubeconfig"
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
            steps{
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'DOCKERHUB_PASS', usernameVariable: 'DOCKERHUB_USER')]) {
                    sh '''
                        echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin
                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
                        docker push ${DOCKER_IMAGE}:latest
                        docker logout
                    '''
                }
            }
        }
        stage('k8s node check'){
            steps{
                withCredentials([file(credentialsId: 'k8s-133', variable: 'KUBECONFIG_FILE_PATH')]) {
                    sh 'cp "$KUBECONFIG_FILE_PATH" "$KUBECONFIG"'
                }
            }
        }
        stage('k8s deploy'){
            steps{
                sh 'kubectl apply -f k8s/deployment.yaml'
            }
        }
    }
    post {
        always {
            sh 'kubectl get pods'
        }
        success {
            sh 'kubectl rollout status deployment/super-project'
        }
        failure {
            sh 'kubectl describe pod -l app=super-project'
        }
    }

}
