pipeline{
    agent any
    stages{
        stage("Sonar Quality Check"){
            agent{
                docker {
                    image 'openjdk:11'
                }
            }
            steps{
                script{
                    withSonarQubeEnv('sonarqubeserver') {
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