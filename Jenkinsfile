pipeline {
   agent any 
    tools {
        maven 'MAVEN3'
        jdk 'JDK17'
    }


    stages {
        stage('Compile et tests') {
            steps {
                echo 'Unit test et packaging'
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
            }
            post {
                always {
                    junit stdioRetention: '', testResults: '**/surefire-reports/*.xml'
                }
                success {
                    archiveArtifacts artifacts: 'application/target/*.jar', followSymlinks: false
                }
                unsuccessful {
                    mail bcc: '', body: 'A corriger ASAP', cc: '', from: '', replyTo: '', subject: 'Build failed', to: 'david.thibau@gmail.com' 
                }
            }

             
        }
        stage('Analyse qualité et vulnérabilités') {
            parallel {
                stage('Vulnérabilités') {
                    agent any
                    steps {
                        echo 'Tests de Vulnérabilités OWASP'
                        sh './mvnw -DskipTests verify'
                    }
                    
                }
                 stage('Analyse Sonar') {
                    agent any
                    environment {
                        SONAR_TOKEN = credentials('SONAR_TOKEN')
                    }
                    steps {
                        echo 'Analyse sonar'
                        sh './mvnw -Dsonar.login=${SONAR_TOKEN} clean integration-test sonar:sonar'
                     }
                    
                }
            }
            
        }
            
        stage('Déploiement intégration') {

            steps {
                echo "Déploiement intégration"
                
            }
        }

     }
    
}

