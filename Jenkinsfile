pipeline { 
    parameters {
        string(name: 'registry', defaultValue: '191124798140.dkr.ecr.us-east-2.amazonaws.com', description: 'Enter the ARN of the registry')
        string(name: 'registry_name', defaultValue: 'flask_app', description: 'Enter the Name of the registry')
        string(name: 'dockerImage', defaultValue: '191124798140.dkr.ecr.us-east-2.amazonaws.com', description: 'Enter the Name of the Docker Image')
        string(name: 'branch', defaultValue: 'dev', description: 'Enter the Name of the branch')
    }
    agent any 
    stages { 
        stage('Cloning Code Repo') { 
            steps { 
                dir("code") { 
                    git branch: branch, 
                    credentialsId: 'ssh_git', 
                    url: 'git@github.com:hamzamaan/Saelor-Application.git'
                }
            }
        }

        stage('Cloning Kubernetes Repo') { 
            steps { 
                dir("kubernetes") { 
                    git branch: branch, 
                    credentialsId: 'ssh_git', 
                    url: 'git@github.com:hamzamaan/Kubernetes_app.git'
                }
            }
        }

//         stage ('Functional Testing'){
//              steps {
//                 sh '''
//                 cd code
//                 cd test
//                 pip3 install -r requirements.txt
//                 python3 -m pytest --junit-xml=result.xml
//                 '''
//             }
//             post {
//                 always {
//                     junit '**/code/test/result.xml'
//                 }
//             }
//         }
//         stage ('OWASP Dependency-Check Vulnerabilities') {
//             steps {
//                 dependencyCheck additionalArguments: ''' 
//                     -o "./code" 
//                     -s "./code"
//                     -f "ALL" 
//                     --prettyPrint''', odcInstallation: 'OWASP-Dependency-Check'

//                 dependencyCheckPublisher pattern: 'dependency-check-report.xml'
//                 sh '''
//                 cd code
//                 curl --location --request POST 'http://3.144.3.241:8080/api/v2/import-scan/' --header 'Authorization: Token d654dc00c584932942be6c064923543b996201fe' --form 'scan_type="Dependency Check Scan"' --form 'engagement="2"' --form 'file=@/home/new_home/workspace/AWS-EKS-Microservice/code/dependency-check-report.xml'
//                 '''
//             }
//         }      
        stage('Code Analysis - SONARQUBE') {
            steps {
                script{
                    env.sonarHome= tool name: 'sonarqube', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                }
                withSonarQubeEnv('sonarqube') {
                    sh """
                      ${sonarHome}/bin/sonar-scanner \
                      -Dsonar.projectKey=Saelor-Application  \
                      -Dsonar.sources=. \
                    """
                }
            }
        }

//         stage('DockerFile Liniting - HADOLINT ') {
//             steps {
//                 sh '''
//                 cd code && docker run --rm -i -v ${PWD}/.hadolint.yml:/.hadolint.yaml hadolint/hadolint hadolint  -f json - < Dockerfile > hadolint.json
                
//                 curl -X POST "http://18.216.55.201:8080/api/v2/reimport-scan/" -H  "accept: application/json" -H  "Content-Type: multipart/form-data" -H  "Authorization: Token 494d5bcaf4cbadcb1f09df5f76f0a4389aaf5ad7"  -F "minimum_severity=Info" -F "active=true" -F "verified=true" -F "scan_type=Hadolint Dockerfile check" -F "push_to_jira=false" -F "close_old_findings=true" -F "file=@/home/new_home/workspace/AWS-EKS-Microservice/code/hadolint.json" -F "product_name=saelor-microservices" -F "scan_date=2022-06-14" -F "engagement_name=saelor-microservices-lint-eg " -F "engagement=6"
//                 '''
//                 // curl --location --request POST 'http://3.144.3.241:8080/api/v2/import-scan/' --header 'Authorization: Token d654dc00c584932942be6c064923543b996201fe' --form 'scan_type="Hadolint Dockerfile check"' --form 'engagement="2"' --form 'file=@/home/new_home/workspace/AWS-EKS-Microservice/code/hadolint.json'

//             }
//         }

        stage('Building Docker image - DOCKER PLUGIN') { 
            steps {
                script {
                    dockerImage = docker.build (registry + ":$BUILD_NUMBER", "./code") 
                }           
            }
        }
        
//         stage ('Docker Image Vulnerability Scan - TRIVY '){
//             steps{
//                 script{
//                     sh '''
//                     trivy image -f json -o results.json --exit-code 0 --severity CRITICAL,HIGH  $registry:$BUILD_NUMBER
//                     curl -X POST "http://18.216.55.201:8080/api/v2/reimport-scan/" -H  "accept: application/json" -H  "Content-Type: multipart/form-data" -H  "Authorization: Token 494d5bcaf4cbadcb1f09df5f76f0a4389aaf5ad7" -F "minimum_severity=Info" -F "active=true" -F "verified=true" -F "scan_type=Trivy Scan" -F "push_to_jira=false" -F "close_old_findings=true" -F "file=@/home/new_home/workspace/AWS-EKS-Microservice/code/results.json" -F "product_name=saelor-microservices" -F "scan_date=2022-06-14" -F "engagement_name=saelor-microservices-eg" -F "engagement=5"

//                     '''
//                     //curl --location --request POST 'http://3.144.3.241:8080/api/v2/import-scan/' --header 'Authorization: Token d654dc00c584932942be6c064923543b996201fe' --form 'scan_type="Trivy Scan"' --form 'engagement="2"' --form 'file=@/home/new_home/workspace/AWS-EKS-Microservice/code/results.json'

//                 }
//             }
//         }
        
        stage('Kubescape Scan'){
            steps{
                script{
                    sh '''
                       BIN_DIR=$JENKINS_HOME/.local/bin/kubescape
                       

                       $BIN_DIR  scan --format junit --output kubescape-results.xml /home/new_home/workspace/AWS-EKS-Microservice/code/*
                       $BIN_DIR scan /home/new_home/workspace/AWS-EKS-Microservice/code/* --submit --account=6e07cf7d-26ec-45bf-a94e-5a48081a630c

                    '''
                
                    
                }
              }
        }
 
        stage('Push Docker image to Registry - DOCKERHUB/ECR/ACR') { 
            steps { 
                script { 
                    sh '''#!/bin/bash
                    aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin $registry
                    docker tag $registry:$BUILD_NUMBER $registry/$registry_name:$BUILD_NUMBER
                    docker push $registry/$registry_name:$BUILD_NUMBER
                    '''
                } 
            }
        }

        stage('Docker Image Tag update') { 
            steps { 
                sh "cd kubernetes/kubernetes && kustomize edit set image $registry/$registry_name:$BUILD_NUMBER" 
            }
        }

        stage('Update Kubernetes Repo to Trigger Argo CD ') { 
            steps { 
                sshagent(['ssh_git']) {
                    sh "cd kubernetes git pull"
                    sh "cd kubernetes  && git commit -am 'Publish new version' || echo 'no changes'"
                    sh "cd kubernetes  && git push --set-upstream origin $branch"
                    echo "$registry/$registry_name:$BUILD_NUMBER"
                }
            }
        }

        stage('Delete Local image locally') { 
            steps { 
                sh "docker rmi $registry:$BUILD_NUMBER"
                sh "docker rmi $registry/$registry_name:$BUILD_NUMBER"
            }
        } 
    }
}
