pipeline {
   agent none 
    tools {
        maven 'MAVEN3'
        jdk 'JDK17'
    }


    stages {
        stage('Compile et tests') {
            agent any
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
                    dir('application/target') {
                        stash includes: '*.jar', name: 'app'
                    }    
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
//                        sh './mvnw -DskipTests verify'
                    }
                    
                }
                 stage('Analyse Sonar') {
                    agent any
                    environment {
                        SONAR_TOKEN = credentials('SONAR_TOKEN')
                    }
                    steps {
                        echo 'Analyse sonar'
//                        sh './mvnw -Dsonar.login=${SONAR_TOKEN} clean integration-test sonar:sonar'
                     }
                    
                }
            }
            
        }
            
        stage('Déploiement intégration') {
           /* when {
                branch 'master'
                beforeOptions true
                beforeInput true
                beforeAgent true
            }*/

            agent any
            input {
                message 'Voulez vous déployer vers les datacenters ?'
                ok 'Déployer'
            }

            steps {
                echo "Déploiement intégration vers les datacenters"
                unstash 'app'
                script {
                    def jsonData = readJSON file: './deployment.json'
                    echo "jsonData ${jsonData}"
                    def integrationUrl = jsonData.integrationURL
                    def dataCenters = jsonData.dataCenters
                    echo "integration ${integrationUrl}"
                    echo "dataCenters ${dataCenters}"
                    for (def dataCenter in dataCenters) {
                        sh "cp *.jar ${integrationUrl}/${dataCenter}.jar"   
                    }
                }
            }
        }
     }
    
}

