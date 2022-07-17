pipeline {
    agent any
    tools {
      maven 'Maven'
        }
    environment {
        MAVEN_CLI_OPTS = '-s .ci-settings.xml --batch-mode --errors --fail-at-end --show-version'
        VERSIONLABELMETHOD = 'OnlyIfThisCommitHasVersion'
        AWS_REGION = 'ap-south-1'
        registryId = '658118898079.dkr.ecr.ap-south-1.amazonaws.com'
        dockerImage = 'testservice'
        dockerRegistry = '658118898079.dkr.ecr.ap-south-1.amazonaws.com/testservice'
    }
    stages {
        stage('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                '''
                }
            }
        stage('Compile') {
            steps {
                sh "mvn ${MAVEN_CLI_OPTS} -DskipTests clean compile"
                archiveArtifacts artifacts: 'target/'
                }
            }
        stage('Test') {
            steps {
                sh "mvn ${MAVEN_CLI_OPTS} test verify"
                archiveArtifacts artifacts: 'target/'
            }
        }
//         stage('sonarqube') {
//             steps {
//                 withSonarQubeEnv(credentialsId: 'sonar.lendpt.app', installationName: 'SonarServer'){
//                     sh "mvn ${MAVEN_CLI_OPTS} -DskipTests sonar:sonar -Dsonar.qualitygate.wait=true"
//                     archiveArtifacts artifacts: 'target/'
//                 }
//             }
//         }
        stage('Build') {
            steps {
                sh "mvn ${MAVEN_CLI_OPTS} jib:build -Dimage.version=latest"
            }
        }
        stage('Build and Push Container Image.') {
            steps {
                script {
                    sh "aws ecr get-login-password --region ${env.AWS_REGION} | docker login --username AWS --password-stdin ${env.registryId}"
                    sh "docker build -t ${env.dockerImage} . --no-cache"
                    sh "docker tag ${env.dockerImage}:latest ${env.dockerRegistry}:${env.BUILD_NUMBER}"
                    sh "docker push ${env.dockerRegistry}:${env.BUILD_NUMBER}"
                }
            }
        }
    }
}
