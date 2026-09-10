pipeline {

    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    }

    stages {

        stage("sonar quality check") {

            agent {
                docker {
                    image 'eclipse-temurin:11'
                }
            }

            steps {

                script {

                    withSonarQubeEnv(credentialsId: 'sonar-token') {

                        sh '''
                            export GRADLE_USER_HOME="$WORKSPACE/.gradle"
                            export SONAR_USER_HOME="$WORKSPACE/.sonar"

                            mkdir -p "$GRADLE_USER_HOME"
                            mkdir -p "$SONAR_USER_HOME/cache/_tmp"

                            chmod +x gradlew

                            ./gradlew sonarqube --stacktrace --info
                        '''
                    }
                    timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }     
                    }
                }
            }
        }
        stage ("docker build & docker push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_pass')]) {
                             sh '''
                                docker build -t 3.110.82.131:8083/springapp:${VERSION} .
                                echo "$docker_pass" | docker login 3.110.82.131:8083 -u admin --password-stdin
                                docker push 3.110.82.131:8083/springapp:${VERSION}
                                docker rmi 3.110.82.131:8083/springapp:${VERSION}

                            '''
                    }
                }    
            }        

        }
       
    }
    post {
        always {
            mail bcc: '', body: "<br>Project: ${env.Job_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}",
        }
    }
}
