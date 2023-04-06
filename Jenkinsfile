pipeline {
    agent any
    environment {
        DOCKERHUB_USERNAME = "krrish1110"
        APP_NAME = "gitops-demo-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}" + "/" + "${APP_NAME}"
        REGISTRY_CREDS = 'dockerhub'
        }
    stages {
        stage('Cleanup Workspace'){
            steps {
                script {
                    cleanWs()
                }
            }
        }
        stage('Checkout Application Source Code GitHub repo'){
            steps {
                git url: 'https://github.com/krrish1110/gitops-demo.git',
                branch: 'master'
            }
        }
        stage('Build Docker Image'){
            steps {
                script{
                    docker_image = docker.build "${IMAGE_NAME}"
                }
            }
        }
        stage('Push Docker Image'){
            steps {
                script{
                    docker.withRegistry('', REGISTRY_CREDS ){
                        docker_image.push("${BUILD_NUMBER}")
                    }
                }
            }
        } 
        stage('Delete Docker Images'){
            steps {
                sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Checkout eks manifest GitHub repo'){
            steps {
                git url: 'https://github.com/krrish1110/gitops-demo-config.git',
                branch: 'master'
            }
        }

        stage('Updating Kubernetes deployment file'){
            steps {
                sh "cat deployment.yml"
                sh "sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yml"
                sh "cat deployment.yml"
            }
        }
        stage('Push the changed deployment file to Git'){
            steps {
                script{
                    withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'pass', usernameVariable: 'user')]) {
                        sh "git config user.email krrish.kj1110@gmail.com"
                        sh "git config user.name krrish1110"
                        //sh "git switch master"
                        sh "git add deployment.yml"
                        sh "git commit -m 'Updated the deployment file'"
                        sh "git push git push https://$user:$pass@github.com/krrish1110/gitops-demo-config.git HEAD:master"
                    }
                }
            }
        }
    }
}


// stage('Build Docker Image'){
//             steps {
//                 sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
//                 sh "docker build -t ${IMAGE_NAME}:latest ."
//             }
//         }
//         stage('Push Docker Image'){
//             steps {
//                 withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'pass', usernameVariable: 'user')]) {
//                     sh "docker login -u $user --password $pass"
//                     sh "docker push ${IMAGE_NAME}:${IMAGE_TAG} ."
//                     sh "docker push ${IMAGE_NAME}:latest ."
//                 }
//             }
//         }
