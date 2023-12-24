//Jenkins File

pipeline{
    agent any
    
    stages {
        stage("Code"){
            steps{
                echo "Cloning the code"
                git url: "https://github.com/Gaurav0807/Django_App.git", branch: "main"
            }
        }
        stage("build"){
            steps{
                echo "Building the image"
                sh "docker build -t my-notes-app ."
            }
        }
        stage("Push to Docker Hub"){
            steps{
                echo "Pushing the image to docker hub"
                withCredentials([usernamePassword(credentialsId:"dockerhub",passwordVariable:"dockerhubPass",usernameVariable:"dockerhubUser")]){
                 sh "docker tag my-notes-app ${env.dockerhubUser}/my-notes-app:latest"
                 sh "docker login -u ${env.dockerhubUser} -p ${env.dockerhubPass}"
                 sh "docker push ${env.dockerhubUser}/my-notes-app:latest"
                }
            }
        }
        stage("Deploy"){
            steps{
                echo "Deploying the container"
                sh "docker-compose down && docker-compose up -d"
            }
        }
    }
}
