import java.util.concurrent.TimeUnit

@Library('jenkins-build-utils')_

def utils = new com.pureport.Utils()

pipeline {
    agent any
    options {
        disableConcurrentBuilds()
    }
    environment {
        DOCKER_REPO = 'docker.dev.pureport.com'
        DOCKER_IMAGE_NAME = 'pureport/nexus-repository-apt'
        VERSION = getVersion()
    }
    stages {
        stage('Build') {
            steps {
                script {
                    sh "docker build -t ${env.DOCKER_IMAGE_NAME}:${env.VERSION} ."
                }
            }
        }
        stage('Publish Docker image') {
            when {
                anyOf {
                    branch 'master'
                }
            }
            steps {
                script {
                    echo "Tagging ${env.DOCKER_IMAGE_NAME}:${env.VERSION} as ${env.DOCKER_REPO}/${env.DOCKER_IMAGE_NAME}:${env.VERSION}..."
                    sh "docker tag ${env.DOCKER_IMAGE_NAME}:${env.VERSION} ${env.DOCKER_REPO}/${env.DOCKER_IMAGE_NAME}:${env.VERSION}"

                    echo "Pushing ${env.DOCKER_REPO}/${env.DOCKER_IMAGE_NAME}:${env.VERSION}..."
                    sh "docker push ${env.DOCKER_REPO}/${env.DOCKER_IMAGE_NAME}:${env.VERSION}"
                }
            }
        }
    }
    post {
        success {
            slackSend(color: '#30A452', message: "SUCCESS: <${env.BUILD_URL}|${env.JOB_NAME}#${env.BUILD_NUMBER}>")
        }
        unstable {
            slackSend(color: '#DD9F3D', message: "UNSTABLE: <${env.BUILD_URL}|${env.JOB_NAME}#${env.BUILD_NUMBER}>")

            script {
                utils.sendUnstableEmail()
            }
        }
        failure {
            slackSend(color: '#D41519', message: "FAILED: <${env.BUILD_URL}|${env.JOB_NAME}#${env.BUILD_NUMBER}>")
            script {
                utils.sendFailureEmail()
            }
        }
        cleanup {
            echo "Removing docker images from local docker registry"
            sh "docker rmi ${env.DOCKER_REPO}/${env.DOCKER_IMAGE_NAME}:${env.VERSION} || true"
            sh "docker rmi ${env.DOCKER_IMAGE_NAME}:${env.VERSION} || true"
        }
    }
}

String getVersion() {
    def props = readProperties(file: 'version.properties')
    return env.BRANCH_NAME == "master"
            ? props.version
            : (props.version + '-' + currentTime())
}

String currentTime() {
    return new Date().format('YYYYMMddhhmmss',TimeZone.getTimeZone('America/New_York'))
}
