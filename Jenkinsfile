pipeline{
    agent any

    options{
        disableConcurrentBuilds
    }

    environment {
        IMAGE_NAME = "venkatesh990/multibranch-flask-app"
        GIT_USER  = "venkatesh-thom"
        GIT_EMAIL = "tvenky359@gmail.com"
    }

    stages {
        stage ("Checkout"){
            steps{
                checkout scm 
            }
        }

        stage ("Build and  push Image") {
            when { branch "main" }

            steps{
                def IMAGE_TAG = "build-${BUILD_NUMBER}"

                withCredentials([usernamePassword(
                    credentialsId: "dockerhub-creds",
                    usernameVariable: "DOCKER_USER"
                    passwordVariable: "DOCKER_PASS"
                )]){
                    sh """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    """


                }
                env.IMAGE_TAG = IMAGE_TAG

            }
        }

        stage ("Update k8 manifest"){
            when {  branch "main" }
            steps{
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'github-creds',
                        usernameVariable: 'GIT_USERNAME',
                        passwordVariable: 'GIT_TOKEN'
                    )])
                    sh """
                    set -e 
                    git config user.name "$GIT_USER"
                    git config user.email "$GIT_EMAIL"
                    
                    git fetch origin
                    git checkout main
                    git reset --hard origin/main

                    sed -i "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|" k8s/deployment.yml
                    git add k8s/deployment.yml
                    git diff --cached --quiet || git commit -m "Updated image to ${IMAGE_TAG}"
                    git push https://${GIT_USERNAME}:${GIT_TOKEN}@github.com/venkatesh-thom/Multi-Branch-Prod.git main
                    """
                }
            }
        }

    }







}