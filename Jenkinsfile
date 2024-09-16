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
        
        stage('clear container') {
            steps {
                script {
                    sh'''
                     docker rm ESPcontainer -f
                     '''
                }
            }
        } 
        stage('clean workspace') {
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
        stage('MVN SONARQUBE'){
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
         stage('run container') {
            steps {
                script {
                    sh'''
                     docker run -itd --name ESPcontainer -v ~/workspace/artifact-test:/project espressif/idf:release-v5.0
                    '''
                }
            }
        }     
       stage('set-up & build') {
            steps {
                script {
                    sh'''
                     docker exec ESPcontainer sh -c "cd /project && . /opt/esp/idf/export.sh && idf.py build"
                     '''
                }
            }
        } 
    

        stage('nexus') {
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
