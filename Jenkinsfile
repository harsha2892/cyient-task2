pipeline {
    agent any
    tools { 
        maven 'M3' 
    }
    options {
        timestamps()
        disableConcurrentBuilds abortPrevious: true
        buildDiscarder logRotator( 
            daysToKeepStr: '2', 
            numToKeepStr: '2'
        )
    }
    environment {
        DOCKERHUB_USERNAME = "harsha2893"
        JOB_NAME = "spring-petclinic"
        APP_NAME = "spring-petclinic"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}" + "/" + "${APP_NAME}"
        REGISTRY_CREDS = 'dockerhub'
    }
    stages {
        stage('Cleanup Workspace') {
            steps {
                script {
                    cleanWs()
                    sh """
                    echo "Cleaned Up Workspace for ${JOB_NAME}"
                    """
                }
            }
        }
        stage('Checkout SCM'){
            steps {
                git url: 'https://github.com/harsha2892/cyentassignment.git',
                branch: 'main'
            }
        }
        stage('Code Build') {
            steps {
                dir('task2and3') {
                    sh "mvn -Dmaven.test.failure.ignore=true clean package"
                }    
            }
            post {
                success {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        stage('Build Docker Image'){
            steps {
                dir('task2and3') {
                    script{
                        docker_image = docker.build "${IMAGE_NAME}"
                    }
                }
            }
        }
        stage('Push Docker Image'){
            steps {
                dir('task2and3') {
                    script{
                        docker.withRegistry('', REGISTRY_CREDS ){
                            docker_image.push("${BUILD_NUMBER}")
                            docker_image.push('latest')
                        }
                    }
                }
            }
        }
         stage('Deploy to Minikube'){
            steps {
                dir('task2and3') {
                    sh 'envsubst < deployment.yml | kubectl apply -f -'
                }    
            }
        }  
        stage('Start Minikube') {
            steps {
                sh "minikube start"
                sh "kubectl get services petclinic-minikube"
            }
        }
    }   
}
