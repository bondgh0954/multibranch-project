pipeline {

    agent any

    tools {
        maven 'maven'
    }

    stages{

        stage('version increment'){
            steps{
                script{
                    echo 'incrementing application version dynamically.........'
                    sh 'mvn build-helper:parse-version versions:set \
                       DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                       versions:commit'
                    def matcher = readFile('pom.xml') ~= '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }

        stage('build jar'){
            steps{
                script{
                    echo 'building application jar file .........'
                    sh ' mvn clean package'
                }
            }
        }

        stage('build image'){
            steps{
                script{
                    echo 'building application docker image .......'
                    sh "docker build -it nanaot/java-app:$IMAGE_NAME ."
                }
            }
        }

        stage('push image'){
            steps{
                script{
                    echo 'pushing image to private registry........'
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]){
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        sh "docker push nanaot/java-app:$IMAGE_NAME"
                    }
                }
            }
        }
    }
}