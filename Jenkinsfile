pipeline {
    agent any
    environment {
        IDF_PATH = '/home/raed/esp/esp-idf'
        IDF_TOOLS_PATH = '/home/raed/.espressif'
        NEXUS_URL = 'http://192.168.33.3:8081/repository/RaedRepo/'
        SONAR_HOST_URL = 'http://192.168.33.3:9000'
        SONAR_TOKEN = 'sqp_ac4e7107d10c6b89a836534e09956899eda9eef7'
    }
    

    stages {
        stage('Clean Container & Workspace') {
            steps {
                script {
                    sh'''
                     docker rm ESPcontainer -f
                     sudo rm -rf ./*
                     '''
                }
            }
        } 
        
        stage('GIT') {
            steps {
                echo 'Pulling from GIT';
                git branch: 'master',
                url: 'https://github.com/K-raed/esp.git'
            }   
        }
        
         stage('Run Container') {
            steps {
                script {
                    sh'''
                     docker run -itd --name ESPcontainer -v ~/workspace/esp_master:/project espressif/idf:release-v5.0
                    '''
                }
            }
        }     
       stage('Set-up & Build') {
            steps {
                script {
                    sh'''
                     docker exec ESPcontainer sh -c "cd /project && . /opt/esp/idf/export.sh && idf.py build"
                     '''
                }
            }
        } 
       stage('SONARQUBE'){
        steps{
            script{
                // Configure SonarQube
                    def scannerHome = tool 'sonar'
                    withSonarQubeEnv('sonar') {
                        sh "${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=proj1 \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=${SONAR_HOST_URL} \
                            -Dsonar.login=${SONAR_TOKEN}"
                    }
                }
             }
         }
        stage('Flawfinder') {
            steps {
                sh 'flawfinder --quiet --html main > flawfinder-report.html'
            }
        }
        stage('Cppcheck') {
            steps {
                sh 'cppcheck --enable=all main 2> cppcheck.html'
            }
        }
 
        stage('NEXUS') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'nexus-credentials-id', passwordVariable: 'NEXUS_PASSWORD', usernameVariable: 'NEXUS_USERNAME')]) {
                        sh '''
                        docker exec ESPcontainer sh -c "curl -u ${NEXUS_USERNAME}:${NEXUS_PASSWORD} \
                            --upload-file /project/build/mqtt_ssl.bin \
                            ${NEXUS_URL}mqtt_ssl.bin"
                        '''
                }       }
            }
        } 
        stage ('MQTT'){
            steps {
                script {
                  sh '''
                    mosquitto_pub -h ceb0a78c.ala.eu-central-1.emqxsl.com -p 8883 -u raedkorbi -P @eNRri#DvGv4Ah9 -t fota -m 'firmware is ready to upload'
                    '''  
                }
            }
        }
                stage('Ngrok 3min Tunnel') {
            steps {
                script {
                    // Start ngrok in the background and get its process ID (PID)
                    def ngrokPid = sh(script: 'ngrok http --url=ghost-holy-radically.ngrok-free.app 8081 & echo $!', returnStdout: true).trim()
                   
                    sleep(180)

                    sh "kill -9 ${ngrokPid}"
                    echo "Ngrok tunnel closed."
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed.'
            publishHTML(target: [
                allowMissing: false,
                keepAll: true,
                reportDir: '.',
                reportFiles: 'flawfinder-report.html',
                reportName: 'Flawfinder Report'
            ])
            publishHTML (target: [
                reportDir: '.',
                reportFiles: 'cppcheck.html',
                reportName: 'Cppcheck Report'
            ])
        }
        success {
            echo 'Pipeline succeeded.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
