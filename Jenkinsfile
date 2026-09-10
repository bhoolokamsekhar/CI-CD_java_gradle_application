pipeline {

    agent any

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
    }
}