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
                       -versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
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
                    sh "docker build -t nanaot/java-app:$IMAGE_NAME ."
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
        stage('deploy Application'){
            steps{
                script{
                    echo 'deploying application.......'
                    def docker = "docker run -p 8080:8080 -d nanaot/java-app:$IMAGE_NAME"
                    sshagent(['key-pair']) {
                        sh "ssh -o StrickHostKeyChecking=no ec2-user@18.194.242.250 ${docker}"
  
                    }
                    
                }
            }
        }
        stage('commit versionchanges'){
            steps{
                script{
                    echo 'updating version increment to git repo......'
                    withCredentials([usernamePassword(credentialsId: 'github-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]){
                        sh "git remote set-url origin https://${PASS}:${PASS}@github.com/bondgh0954/multibranch-project.git"

                        sh 'git config --global user.name "Jenkins"'
                        sh ' git config --global user.email "jenkins@example.com"'

                        sh 'git add .'
                        sh 'git commit -m "commit changes"'
                        sh 'git push origin HEAD:main'


                    }
                }
            }
        }
    }
}