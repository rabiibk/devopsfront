pipeline {
    agent any

    environment {
        NEXUS_URL = 'http://192.168.12.150:8081'
        NEXUS_REPO = 'repository/maven-releases'
        ARTIFACT_GROUP = 'tn.example'
        ARTIFACT_NAME = 'angular-final'
        ARTIFACT_VERSION = "1.0.${env.BUILD_NUMBER}"
        DOCKER_REPO = 'rabii1990/frontend'
        DOCKER_IMAGE_TAG = 'latest'
    }

    stages {
        stage('Git') {
            steps {
                echo 'My first job pipeline angular'
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/rabiibk/devopsfront.git']]])
            }
        }


        stage('Install Dependencies') {
           steps {
           dir('/var/lib/jenkins/workspace/devopsfront'){
                sh 'npm install'
                }
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build --prod'
            }
        }


        stage('Publish to Nexus') {
            steps {
                script {
                    def artifactPath = "dist/${ARTIFACT_NAME}-{ARTIFACT_VERSION}.tar.gz"
                    def nexusArtifactUrl = "${NEXUS_URL}/${NEXUS_REPO}/${ARTIFACT_GROUP}/${ARTIFACT_NAME}/${ARTIFACT_VERSION}/${ARTIFACT_NAME}-${ARTIFACT_VERSION}.tar.gz"

                    // Deploy the artifact to Nexus
                    sh "curl -v --user admin:nexus --upload-file dist/angular-frontend ${nexusArtifactUrl}"
                }
            }
        }

        stage('Pull & Build Docker Image') {
            steps {
                sh "curl -o angular-final.jar ${NEXUS_URL}/${NEXUS_REPO}/${ARTIFACT_GROUP}/${ARTIFACT_NAME}/${ARTIFACT_VERSION}/${ARTIFACT_NAME}-${ARTIFACT_VERSION}.tar.gz" // Télécharger l'archive depuis Nexus

                sh "docker build -t angular-final:latest -f /home/rabii/docker/frontend/Dockerfile /home/rabii/docker/frontend/"
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                sh "docker tag angular-final:latest ${DOCKER_REPO}:${DOCKER_IMAGE_TAG}"
                sh "docker login -u rabii1990 -p rabiiradar2012"
                sh "docker push ${DOCKER_REPO}:${DOCKER_IMAGE_TAG}"
            }
        }
    }

    post {
        always {
            sh 'npm cache clean --force' // Nettoyer le cache npm
        }
    }
}
