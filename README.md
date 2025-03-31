#This is static website with HTML,CSS and ready for deployment
jenkins script
pipeline {
    agent any
    
    tools{
        jdk 'jdk17'
    }
    
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/parnik-code/shopingcartapp.git'
            }
        }
        
        stage('Sonarqube analysis') {
            steps {
                sh '''  $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.host.url=http://15.222.249.40:9000 \
                        -Dsonar.login=squ_c8a953d33928b9aa37c82fa4a414123f525c11b5 \
                        -Dsonar.projectName=shopping-cart \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=shopping-cart
                    '''
            }
        }
        
    stage('docker build') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker build -t parnikdwivedi/shoppingcart .'
                        sh 'docker push parnikdwivedi/shoppingcart'
                    }
                }
            }
        }    
        
    stage('trivy scan') {
            steps {
                sh 'trivy fs --severity HIGH,CRITICAL --format table .'
            }
        }
        
    stage('docker deployement') {
            steps {
                sh 'docker run -d -p 8000:80 parnikdwivedi/shoppingcart'
            }
        }  
    }
}
