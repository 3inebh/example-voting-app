pipeline {

    agent none
    
    stages{
        stage('worker build'){
            agent{
                docker{
                    image 'maven:3.8.3-jdk-8-slim'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when{
                changeset '**/worker/**'
            }
            steps{
                echo 'Compiling worker app'
                dir('worker'){
                    sh 'mvn compile'
                }
            }
        }
        stage('worker test'){
            agent{
                docker{
                    image 'maven:3.8.3-jdk-8-slim'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when{
                changeset '**/worker/**'
            }
            steps{
                echo 'Running unit tests on worker app'
                dir('worker'){
                    sh 'mvn clean test'
                }
            }
        }
        stage('worker package'){
            agent{
                docker{
                    image 'maven:3.8.3-jdk-8-slim'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when{
                branch 'master'
                changeset '**/worker/**'
            }
            steps{
                echo 'Package worker app'
                dir('worker'){
                    sh 'mvn package -DskipTests'
                    archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                }
            }
        }
        stage('worker docker-package'){
            agent any
            when{
                branch 'master'
                changeset '**/worker/**'
            }
            steps{
                echo 'Package worker app with docker'
                script{
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin'){
                        def workerImage = docker.build("zinebh/worker:v${env.BUILD_ID}", "./worker")
                        workerImage.push()
                        workerImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }
        stage('result build'){
            agent{
                docker{
                    image 'node:8.16.0-alpine'
                }
            }
            when{
                changeset '**/result/**'
            }
            steps{
                echo 'Compiling result app'
                dir('result'){
                    sh 'npm install'
                }
            }
        }
        stage('result test'){
            agent{
                docker{
                    image 'node:8.16.0-alpine'
                }
            }
            when{
                changeset '**/result/**'
            }
            steps{
                echo 'Running unit tests on result app'
                dir('result'){
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }
        stage('result docker-package'){
            agent any
            when{
                branch 'master'
                changeset '**/result/**'
            }
            steps{
                echo 'Package result app with docker'
                script{
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin'){
                        def workerImage = docker.build("zinebh/result:v${env.BUILD_ID}", "./result")
                        workerImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }
        stage('vote build'){
            agent{
                docker{
                    image 'python:3.6-alpine3.13'
                    args '--user root'
                }
            }
            when{
                changeset '**/vote/**'
            }
            steps{
                echo 'Compiling vote app'
                dir('vote'){
                    sh 'pip install -r requirements.txt'
                }
            }
        }
        stage('vote test'){
            agent{
                docker{
                    image 'python:3.6-alpine3.13'
                    args '--user root'
                }
            }
            when{
                changeset '**/vote/**'
            }
            steps{
                echo 'Running tests on vote app'
                dir('vote'){
                    sh 'pip install -r requirements.txt'
                    sh 'nosetests -v'
                }
            }
        }
        stage('vote docker-package'){
            agent any
            when{
                branch 'master'
                changeset '**/vote/**'
            }
            steps{
                echo 'Package vote app with docker'
                script{
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin'){
                        def workerImage = docker.build("zinebh/vote:v${env.BUILD_ID}", "./vote")
                        workerImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }
    }
    
    post{
        always{
            echo 'Pipeline is completed'
        }
        failure{
            slackSend (channel: "instavote-ci", message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        success{
            slackSend (channel: "instavote-ci", message: "Build Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
    }
    
}