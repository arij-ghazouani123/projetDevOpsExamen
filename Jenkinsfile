pipeline {
    agent any
        tools{
        nodejs 'DevOpsfrontend'
    } 
    
  
    
    stages {
        stage('Github') {
            steps {
                checkout scm
            }
        }
    
     stage(' Unit Tests - Email') {
            steps {
                dir('DevOps_Project') {
                    script {
                        try {
                            sh 'mvn clean install'
                        } catch (Exception e) {
                            currentBuild.result = 'FAILURE'
                            error("Build failed: ${e.message}")
                        }
                    }
                }
            }
           post {
                success {
                    script {
                        def subject = "TESTES"
                        def body = "SUCCESS"
                        def to = 'arij.ghazouani22@gmail.com'

                        mail(
                            subject: subject,
                            body: body,
                            to: to,
                        )
                    }
                }
                failure {
                    script {
                        def subject = "Build Failure - ${currentBuild.fullDisplayName}"
                        def body = "The build has failed "
                        def to = 'arij.ghazouani22@gmail.com'

                        mail(
                            subject: subject,
                            body: body,
                            to: to,
                        )
                    }
                }
                
            }
        }
        
       stage('SONARQUBE') {
            steps {
                dir('DevOps_Project') {
                sh 'mvn sonar:sonar -Dsonar.host.url=http://localhost:9000 -Dsonar.login=admin -Dsonar.password=azerty123'
            }
            }
       }

         stage('NEXUS') {
            steps {
                dir('DevOps_Project') {
                sh 'mvn clean deploy -DskipTests'
            }
            }
         }

            stage('BUILD FRONT') {
                steps {
                    dir('DevOps_Project_Front') {
                        script {
                            
                            sh 'npm install -g npm@latest'
                            sh 'npm install --force'
                            sh 'npm run build'      
                        }
                    }
                }
            }

      stage('LOGIN DOCKER') {
        steps {
        script {
          withCredentials([string(credentialsId: 'dockerhub-pwd', variable: 'dockerhubpwd')]) {
                sh 'docker login -u docker0922d -p ${dockerhubpwd}'
                }
            }
        }    
      }

        stage('DOCKER IMAGE BACKEND') {
            steps {
                dir('DevOps_Project') {
                    script {
                        sh 'docker build -t docker0922d/devopsbackend .'
                        sh 'docker push docker0922d/devopsbackend'
                    }
                }
            }
        }
        stage('DOCKER IMAGE FRONTEND') {
            steps {
                dir('DevOps_Project_Front') {
                    script {
                        sh 'docker build -t docker0922d/devopsfrontend .'
                        sh 'docker push docker0922d/devopsfrontend'
                        
                    }
                }
            }
        }
        
       stage('DEPLOY APP') {
            steps {
                
                script {
                    sh 'docker-compose -f docker-compose.yml up -d' 
                    sh 'docker-compose -f docker-compose.yml start'                       
                }
                
            }
       }
    }


}