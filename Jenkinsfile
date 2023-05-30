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
        stage("Helm Chart Validation by Datree"){
            steps{
                script{
                    dir('kubernetes/') {
                        withEnv(['DATREE_TOKEN=0b527d74-30a8-4587-a28b-8d572999c0e8']) {
                            sh 'helm datree test myapp/ --ignore-missing-schemas'
                        }
                        
                    }
                }
            }
        }
        stage("Pushing helm chart to nexus") {
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus-docker-pass', variable: 'nexustodocker')]) {
                        dir('kubernetes/') {
                            sh '''
                                helmversion=$(helm show chart ./myapp/ | grep version | cut -d: -f 2 | tr -d " ")
                                tar -czvf myapp-${helmversion}.tgz myapp/ 
                                curl -u admin:$nexustodocker http://192.168.1.16:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v 
                            '''
                        }    
                    }
                    
                }
            }
        }
        stage('Manual Approval'){
            steps{
                script{
                    timeout(10) {
                        mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> Go to build url and approve the deployment <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "ajay_durande@rediffmail.com";
                        input(id: "${VERSION}", message: "Deploy ${params.project_name}?", ok: 'Deploy')
                    }
                }
            }
        }
        stage('Deploying on Kubernetes') {
            steps {
                script{
                    dir ("kubernetes/"){
                        sh 'helm upgrade --install --set image.repository="192.168.1.16:8083/springapp" --set image.tag="${VERSION}" myjavaapp myapp/ ' 
                    }
                }
                
            }
        }
        stage('Verifying app deployment'){
            steps{
                script{
                    timeout(2) {
                        sh 'kubectl run curl --image=curlimages/curl -i --rm --restart=Never -- curl myjavaapp-myapp:80'
                    }
                }
            }
        }
    }
    
    post{
        always{
            mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "ajay_durande@rediffmail.com";
        }
    }    
}