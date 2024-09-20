def integrationUrl
def dataCenters

pipeline {
   agent none 



    stages {
        stage('Compile et tests') {
            agent {
                kubernetes {
                    inheritFrom 'jdk17-agent'
                }
            }
            steps {
                container(name: 'openjdk-17') {
                    echo 'Unit test et packaging'
                    sh './mvnw -Dmaven.test.failure.ignore=true clean package'
                }

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
                    tools {
                        maven 'MAVEN3'
                        jdk 'JDK17'
                    }
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
                    tools {
                        maven 'MAVEN3'
                        jdk 'JDK17'
                }
                    steps {
                        echo 'Analyse sonar'
//                        sh './mvnw -Dsonar.login=${SONAR_TOKEN} clean integration-test sonar:sonar'
                     }
                    
                }
            }
            
        }
        stage('Push vers dockerhub') {
            agent any
            steps {
                echo 'Push vers dockerhub'
                unstash 'app'
                script {
                    def dockerImage = docker.build("dthibau/multi-module",".");
                    docker.withRegistry('https://registry.hub.docker.com', 'dthibau_docker') {
                        dockerImage.push "$BRANCH_NAME"
                    }             
                }
            }
        }
        stage('Read Deployment Info') {
            agent any

            steps {
                echo "Read Deployment Info"
                script {
                    def jsonData = readJSON file: './deployment.json'
                    echo "jsonData ${jsonData}"
                    integrationUrl = jsonData.integrationURL
                    dataCenters = jsonData.dataCenters
                    echo "integration ${integrationUrl}"
                    echo "dataCenters ${dataCenters}"
                 }
                
            }
        }  
        stage('Déploiement vers DATACENTER ??') {
            agent none

            steps {
                input message: "Voulez vous déployer vers $dataCenters", ok: 'Déployer'
               echo "Deploying ..."
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

            steps {
                echo "Déploiement intégration vers les datacenters"
                unstash 'app'
                script {
                    for (def dataCenter in dataCenters) {
                        sh "cp *.jar ${integrationUrl}/${dataCenter}.jar"   
                    }
                }
            }
        }
     }
    
}

