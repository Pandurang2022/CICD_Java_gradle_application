pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("Sonar Quality Check"){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonarqube-token') {
                        sh 'chmod +x gradlew'
                        sh './gradlew sonar'
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
        stage("Docker build and image push") {
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus-docker-pass', variable: 'nexustodocker')]) {
                        sh '''
                            docker build -t 192.168.1.16:8083/springapp:${VERSION} .
                            docker login -u admin -p $nexustodocker 192.168.1.16:8083
                            docker push 192.168.1.16:8083/springapp:${VERSION}
                            docker rmi 192.168.1.16:8083/springapp:${VERSION}
                        '''
                    }
                    
                }
            }
        }
    }    
}