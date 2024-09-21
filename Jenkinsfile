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
        
        stage('Clear Container') {
            steps {
                script {
                    sh'''
                     docker rm ESPcontainer -f
                     '''
                }
            }
        } 
        stage('Clean Workspace') {
            steps {
                script {
                    sh'''
                     sudo rm -rf ./*
                     '''
                }
            }
        } 
        stage('GIT') {
            steps {
                echo 'Pulling from GIT';
                git branch: 'master',
                url: 'https://github.com/K-raed/esp'
            }   
        }
        stage('test SONARQUBE'){
        steps{
            script{
                // Configure SonarQube
                    def scannerHome = tool 'sonar'
                    withSonarQubeEnv('sonar') {
                        sh"${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=proj1 \
                            -Dsonar.organization=sonarsource-cfamily-examples         
                            -Dsonar.sources=. \
                            -Dsonar.host.url=${SONAR_HOST_URL} \
                            -Dsonar.login=${SONAR_TOKEN}"
                     }
                  }
            }
         }
         stage('Run Container') {
            steps {
                script {
                    sh'''
                     docker run -itd --name ESPcontainer -v ~/workspace/artifact-test:/project espressif/idf:release-v5.0
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
        stage('NEXUS') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'nexus-credentials-id', passwordVariable: 'NEXUS_PASSWORD', usernameVariable: 'NEXUS_USERNAME')]) {
                        sh '''
                        docker exec ESPcontainer sh -c "curl -u ${NEXUS_USERNAME}:${NEXUS_PASSWORD} \
                            --upload-file /project/build/blink.bin \
                            ${NEXUS_URL}blink.bin"
                        '''
                }       }
            }
        } 
        stage ('MQTT'){
            steps {
                script {
                  sh '''
                    mosquitto_pub -h 07bfd16103b24e65b1143ae311bf7a31.s1.eu.hivemq.cloud -p 8883 -u raedkorbi -P @eNRri#DvGv4Ah9 -t fota -m 'firmware is ready to upload'
                    '''  
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed.'
        }
        success {
            echo 'Pipeline succeeded.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
