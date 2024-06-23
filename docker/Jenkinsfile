pipeline {
    agent any
    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Pass the name of branch to build from ')
        string(name: 'REPO_URL', defaultValue: 'https://github.com/ityourway/learning-jenkins-repo.git', description: 'Pass the Repository url to build from ')
        }
    environment { 
        BRANCH = "${params.BRANCH}"
        REPO_URL = "${params.REPO_URL}"

    }
    stages {
        stage('Clone GitHub Repo') {
            steps {
                script {
                git branch: "${BRANCH}", credentialsId: 'jenkins-github-creds', url: "${REPO_URL}"
            }
        }
        }
        stage('Building Docker Image') {
            steps {
                script {
             //   withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'jenkins-aws-creds', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                
                dir('docker') {
                 sh "docker build -t oxer-html-image ."
}
                
                 }
                }           
            }
        
        stage('Push To DockerHub Registry') {
            steps {
                script {
               // withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'jenkins-aws-creds', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                withCredentials([usernamePassword(credentialsId: 'jenkins_dockerhub_creds', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                sh """
                docker login --username="$USERNAME" --password="$PASSWORD"
                docker tag oxer-html-image ityourway/oxer-html-repo:v001
                docker push ityourway/oxer-html-repo:v001
                """
}
                
                
}
                }
            }
        stage('Deploy On Remote Server') {
            steps {
                 script {
                  sshPublisher(
                   continueOnError: false, failOnError: true,
                   publishers: [
                    sshPublisherDesc(
                     configName: "test_server",
                     verbose: true,
                     transfers: [
                      sshTransfer(
                       sourceFiles: "",
                       removePrefix: "",
                       remoteDirectory: "",
                       execCommand: """
                          docker stop oxer-app || true
                          docker rm oxer-app || true
                          docker rmi -f ityourway/oxer-html-repo:v001 || true
                          docker run -itd --name oxer-app -p 80:80 ityourway/oxer-html-repo:v001
                          """
                      )
                     ])
                   ])
                 }
                }
            }
        }
    }
