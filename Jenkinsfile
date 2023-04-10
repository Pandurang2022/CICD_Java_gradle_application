pipeline{
    agent any
    stages{
        stage("Sonar Quality Check"){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonarqube-token') {
                        sh 'chmod +x gradlew'
                        sh './gradlew sonar'
                    }
                    
                }
            }
        }
    }
    post{
        always{
            echo "Success"
        }
    }
}