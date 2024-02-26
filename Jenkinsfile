pipeline {
    environment {
        imagerepo = 'YOUR_IMAGE_REPO'
        imagename = 'YOUR_IMAGE_NAME'
    }

    agent any
    
    stages {
        stage('Build Docker Image') {
            steps {
                sh "docker build --no-cache . -t ${imagename}:v${BUILD_NUMBER}"
            }
        }
        
        stage('Tag Docker Image') {
            steps {
                sh "docker tag ${imagename}:v${BUILD_NUMBER} ${imagerepo}/${imagename}:v${BUILD_NUMBER}"
            }
        }
        
        stage('Push Docker Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'YOUR_DOCKER_CREDENTIAL_ID', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    sh "docker push ${imagerepo}/${imagename}:v${BUILD_NUMBER}"
                }
            }
        }
        
        stage('Remove Docker Image') {
            steps {
                sh "docker rmi ${imagename}:v${BUILD_NUMBER}"
                sh "docker rmi ${imagerepo}/${imagename}:v${BUILD_NUMBER}"
            }
        }
        
        stage('Update Manifest') {
            steps {
                sshAgent(credentials: ['YOUR_GITHUB_CREDENTIAL_ID']) {
                    sh """
                        rm -rf node-user-management-deployment
                        git clone https://github.com/YOUR_USERNAME/node-user-management-deployment
                        cd node-user-management-deployment
                        sed -i 's/newTag.*/newTag: v${BUILD_NUMBER}/g' kustomize/overlays/*/*kustomization.yaml
                        git config user.email 'YOUR_EMAIL'
                        git config user.name 'YOUR_GITHUB_USERNAME'
                        git add kustomize/overlays/*/*kustomization.yaml
                        git commit -m 'Update image version to: ${BUILD_NUMBER}'
                        git push origin master -f
                    """
                }
            }
        }
    }

    post { 
        always {
            cleanWs()
        }
    }
}
