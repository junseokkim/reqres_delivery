pipeline {
    agent any

    environment {
        REGISTRY = 'kjs08.azurecr.io'
        IMAGE_NAME = 'delivery'
        AZURE_CREDENTIALS_ID = 'Azure-Cred'
        TENANT_ID = 'f46af6a3-e73f-4ab2-a1f7-f33919eda5ac' // Service Principal 등록 후 생성된 ID
    }
 
    stages {
        stage('Clone Repository') {
            steps {
                checkout scm
            }
        }

        stage('Maven Build') {
            steps {
                withMaven(maven: 'Maven') {
                    sh 'mvn package -DskipTests'
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    imageTag = "v${env.BUILD_NUMBER}"
                    image = docker.build("${REGISTRY}/${IMAGE_NAME}:${imageTag}")
                    
                    sh "az acr login --name ${REGISTRY.split('\\.')[0]}"
                    sh "docker push ${REGISTRY}/${IMAGE_NAME}:${imageTag}"
                }
            }
        }

        stage('Update Deployment YAML for ArgoCD') {
            steps {
                script {
                    sh """
                    sed -i 's|image: .*|image: ${REGISTRY}/${IMAGE_NAME}:v${env.BUILD_NUMBER}|' ./azure/deployment.yaml

                    git config --global user.email "jenkins@mycompany.com"
                    git config --global user.name "Jenkins"
                    git add ./azure/deployment.yaml
                    git commit -m "Update image tag to v${env.BUILD_NUMBER}"
                    git push origin main
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ CI 성공: Docker 이미지 빌드 & GitOps 업데이트 완료"
        }
        failure {
            echo "❌ CI 실패: Docker 빌드 또는 Git 업데이트 실패"
        }
    }
}
