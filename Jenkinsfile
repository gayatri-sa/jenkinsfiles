pipeline {
    agent any
    stages {
        // stage('Step 1') {
        //     steps {
        //         echo 'Hello World'
        //         git url: 'https://github.com/gayatri-sa/samplejava'
        //     }
        // }
        stage('Clone Project') {
            steps {
                git branch: 'master', url: 'https://github.com/gayatri-sa/samplejava'
                stash name:'samplejava', includes:'**/*'
            }
        }
        stage('Build Project') {
            agent {
                // label 'docker'
                docker {
                    image 'maven:3.8.1-adoptopenjdk-11'
                    label 'docker'
                    args  '-v /home/jenkins/.m2:/home/jenkins/.m2'
                }
            }
            // environment {
            //     JAVA_TOOL_OPTIONS = '-Duser.home=/var/maven'
            // }
            steps {
                sh 'rm -fr /home/jenkins/workspace/Sample/*'
                unstash 'samplejava'
                sh 'ls -l /home/jenkins/workspace/Sample'
                // sh 'mvn -Dmaven.repo.local=/home/jenkins/.m2 clean package'
                sh 'mvn clean package'
            }
            post {
                success {
                    stash includes: '**/target/*.war', name: 'app' 
                }
            }
        }
        stage('Build and Push Docker Image') {
            agent {
                label 'docker'
            }
            environment {
                registry = "gayatrisa/samplejava-pipeline-test"
                registryCredential = 'dockerhub'
                dockerImage = ''
            }
            steps {
                sh "find /home/jenkins/workspace/Sample -mindepth 1 -depth -exec rm -rf {} ';'"
                git branch: 'master', url: 'https://github.com/gayatri-sa/jenkins-pipeline'
                sh 'tree ./'
                unstash 'app'
                script {
                  dockerImage = docker.build(registry + ":$BUILD_NUMBER", "-f ./tomcat/Dockerfile ./")
                  docker.withRegistry( '', registryCredential ) {
                    dockerImage.push()
                  }
                }
                sh "docker rmi $registry:$BUILD_NUMBER"
            }
        }
    }
}