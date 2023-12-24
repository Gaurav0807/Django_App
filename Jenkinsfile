//1.) In Jenkins Env Object is automatically provided by jenkins environment, and it contains a set of environment  variables that are available to your pipeline script.
//2.) Parameters block is defined inside the pipeline block in a Jenkinsfile to declare parameters specific to that pipeline. 
//When we define parameters with in pipeline block they are associated  with that particular pipeline.


//Jenkins File
def deploy_code(branch_name){
    product_name ="Todo_app"
    println "Pushing code to s3"

    sh """
        echo 'Start pushing code to s3\n'
        aws s3 sync . "s3://todo-bucket-web-app/$env.ENV_NAME/${product_name}/code/" --region 'us-east-1'
    """
}

//Deploy env. 
def getEnvironmentName(branch) {
  if (branch.toLowerCase() == 'master') {
    return 'prod'
  } else if (branch.toLowerCase() == 'release-qa') {
    return 'qa'
  } else {
    return null
  }
}

pipeline{
    agent any

    parameters{
        booleanParam(name: 's3_deploy', defaultValue: false, description: 'Toggle to decide whether to deploy or not')
    }
    environment{
        ENV_NAME = getEnvironmentName(env.BRANCH_NAME)
    }
    
    stages {
        stage("Code"){
            steps{
                echo "Cloning the code"
                git url: "https://github.com/Gaurav0807/Django_App.git", branch: "main"
            }
        }
        stage("Code Push"){
            when{
                anyOf{
                    allOf{
                        expression{
                            env.BRANCH_NAME.toLowerCase() == 'main
                    }
                        expression{
                            params.s3_deploy == true
                        }
                }
                    allOf{
                        expression{
                            env.BRANCH_NAME.toLowerCase() == 'release-qa'
                        }
                        expression{
                            params.s3_deploy == true
                        }   
                    }
            }
            }
            steps {
                deploy_code(env.BRANCH_NAME)                
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
