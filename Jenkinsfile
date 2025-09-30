pipeline {
    agent {
        docker {
            image 'vaibhavs11/maven-docker:latest'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        SONAR_URL = "http://44.203.136.106:9000"
    }

    stages {

        stage('Build and Test') {
            steps {
                sh 'ls -ltr'
                sh 'cd java-maven-sonar-argocd-helm-k8s/spring-boot-app && mvn clean package'
            }
        }

        stage('Check Java Version') {
            steps {
                sh 'java -version'
                sh 'mvn -version'
            }
        }

        stage('Static Code Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                    dir("java-maven-sonar-argocd-helm-k8s/spring-boot-app") {
                        sh '''
                          echo "Java in use:"
                          java -version
                          echo "Maven in use:"
                          mvn -version
                          mvn clean verify sonar:sonar \
                            -Dsonar.host.url=${SONAR_URL} \
                            -Dsonar.login=$SONAR_AUTH_TOKEN \
                            -Dsonar.projectKey=spring-boot-demo
                        '''
                    }
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-cred',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                      cd java-maven-sonar-argocd-helm-k8s/spring-boot-app
                      docker build -t ${DOCKER_USER}/ultimate-cicd:${BUILD_NUMBER} .
                      echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                      docker push ${DOCKER_USER}/ultimate-cicd:${BUILD_NUMBER}
                    '''
                }
            }
        }

        stage('Update Deployment File') {
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                      git config --global --add safe.directory "/var/lib/jenkins/workspace/Demo Pipeline"
                      git config user.email "vaibhavsawant201@gmail.com"
                      git config user.name "Vaibhav Sawant"

                      git checkout main
                      git pull --rebase origin main

                      cd java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests
                      sed -i "s#image: vaibhavs11/ultimate-cicd:.*#image: vaibhavs11/ultimate-cicd:${BUILD_NUMBER}#g" deployment.yml

                      git add deployment.yml

                      if git diff-index --quiet HEAD --; then
                        echo "No changes to commit"
                      else
                        git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                        git push https://$GITHUB_TOKEN@github.com/vaibhavsawant201/JenkinsTest HEAD:main
                      fi
                    '''
                }
            }
        }
    }
}
