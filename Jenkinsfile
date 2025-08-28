pipeline {
    agent any
    tools {
        maven 'maven'
    }
    environment {
        IMAGE_NAME  ="springbootapp"
        IMAGE_TAG   ="latest"
        // ACR_NAME    ="jenkinsazure"
        // TENANT_ID   ="265ab705-04a7-4f67-af07-dcf0d1388db9"
        // ACR_LOGIN_SERVER ="${ACR_NAME}.azurecr.io"
        // FULL_IMAGE_NAME ="${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG}"
    }
    stages {
        stage('Checkout From Git') {
            steps {
                git branch: 'prod', url: 'https://github.com/fazaluddinsyed/enahanced-petclinc-springboot.git'
            }
        }  
        stage('Maven Validate') {
            steps {
                echo "This is Maven Validate Stage"
                sh 'mvn validate'
            }
        }   
        stage('Maven Compile') {
            steps {
                echo "This is Maven Compile Stage"
                sh 'mvn compile'
            }
        } 
        stage('Sonar Analysis') {
            environment {
                SCANNER_HOME = tool 'Sonar-scanner'
            }
            steps {
                withSonarQubeEnv('sonarserver') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.organization=fazaluddin08 \
                        -Dsonar.projectName=SpringBootPet \
                        -Dsonar.projectKey=fazaluddinsyed_enahanced-petclinc-springboot \
                        -Dsonar.java.binaries=./target
                    '''
                }
            }
        }
         stage('Maven Package') {
            steps {
                echo "This is Maven Package Stage"
                sh 'mvn package'
            }
        }
        stage('Build + Test + Sonar (Maven)') {
            steps {
                withSonarQubeEnv('sonarserver') {
                    sh '''
                        mvn -B clean \
                          org.jacoco:jacoco-maven-plugin:prepare-agent \
                          verify sonar:sonar \
                          -Dsonar.organization=fazaluddin08 \
                          -Dsonar.projectKey=fazaluddinsyed_enahanced-petclinc-springboot \
                          -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
                    '''
                }
            }
        } 
        stage('Sonar Quality Gate'){
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true, credentialsId: 'sonar-token'
                }
            }
        }
        stage ('Docker Build'){
            steps {
                script {
                    echo 'Docker Build Started'
                    docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }
        // stage ('ACR LOGIN'){
        //     steps {
        //         withCredentials([usernamePassword(credentialsId: 'azure-acr-sp', usernameVariable: 'AZURE_USERNAME',passwordVariable: 'AZURE_PASSWORD')]){
        //             script {
        //                 echo "Azure login to container registry"
        //                 sh '''
        //                 az login --service-principal -u $AZURE_USERNAME -p $AZURE_PASSWORD --tenant $TENANT_ID
        //                 az acr login --name $ACR_NAME
        //                 '''
        //             }
        //         }
        //     }
        // }
        // stage('Docker Push to ACR'){
        //     steps {
        //         script {
        //             echo "Docker Push image to Registry" 
        //             sh '''
        //             docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${FULL_IMAGE_NAME}
        //             docker push ${FULL_IMAGE_NAME}
        //             '''
        //         }
        //     }
        // }
    }
}