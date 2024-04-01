@Library('my-shared-library') _

pipeline{

    agent any
    tools {
        maven 'Maven 3.9.6'
        jdk 'jdk-17'
    }

    parameters{

        choice(name: 'action', choices: 'create\ndelete', description: 'Choose create/Destroy')
        string(name: 'ImageName', description: "name of the docker build", defaultValue: 'javapp')
        string(name: 'ImageTag', description: "tag of the docker build", defaultValue: 'v1')
        string(name: 'DockerHubUser', description: "name of the Application", defaultValue: 'vikashashoke')
    }

    stages{
         
        stage('Git Checkout'){
                    when { expression {  params.action == 'create' } }
            steps{
            gitCheckout(
                branch: "main",
                url: "https://github.com/Awsbarath/java17.git"
            )
            }
        }
        stage('Unit Test maven'){
         
         when { expression {  params.action == 'create' } }

            steps{
               script{
                   
                   mvnTest()
               }
            }
        }
        stage('Integration Test maven'){
         when { expression {  params.action == 'create' } }
            steps{
               script{
                   
                   mvnIntegrationTest()
               }
            }
        }
        stage('Static code analysis: Sonarqube'){
         when { expression {  params.action == 'create' } }
            steps{
               script{
                   
                   def SonarQubecredentialsId = 'credentialsId'
                   statiCodeAnalysis(SonarQubecredentialsId)
               }
            }
        }
        stage('Quality Gate Status Check : Sonarqube'){
         when { expression {  params.action == 'create' } }
            steps{
               script{
                   
                   def SonarQubecredentialsId = 'credentialsId'
                   QualityGateStatus(SonarQubecredentialsId)
               }
            }
        }
        stage('Maven Build : maven'){
         when { expression {  params.action == 'create' } }
            steps{
               script{
                   
                   mvnBuild()
               }
            }
        }
        stage('Docker Image Build : ECR'){
          when { expression {  params.action == 'create' } }
             steps{
                script{
                   
                    dockerBuild("${params.aws_account_id}","${params.Region}","${params.ECR_REPO_NAME}")
                }
             }
         }
        stage('Docker Image Scan: trivy '){
          when { expression {  params.action == 'create' } }
             steps{
                script{
                   
                    dockerImageScan("${params.aws_account_id}","${params.Region}","${params.ECR_REPO_NAME}")
                }
             }
         }
        stage('Docker Image Push : ECR '){
          when { expression {  params.action == 'create' } }
             steps{
                script{
                   
                    dockerImagePush("${params.aws_account_id}","${params.Region}","${params.ECR_REPO_NAME}")
                }
             }
         }   
        stage('Docker Image Cleanup : ECR '){
          when { expression {  params.action == 'create' } }
             steps{
                script{
                   
                    dockerImageCleanup("${params.aws_account_id}","${params.Region}","${params.ECR_REPO_NAME}")
                }
             }
         } 
        stage('Create EKS Cluster : Terraform'){
            when { expression {  params.action == 'create' } }
            steps{
                script{

                    dir('eks_module') {
                      sh """
                          
                          terraform init 
                          terraform plan -var 'access_key=$ACCESS_KEY' -var 'secret_key=$SECRET_KEY' -var 'region=${params.Region}' --var-file=./config/terraform.tfvars
                          terraform apply -var 'access_key=$ACCESS_KEY' -var 'secret_key=$SECRET_KEY' -var 'region=${params.Region}' --var-file=./config/terraform.tfvars --auto-approve
                      """
                  }
                }
            }
        }
        /*
        stage('Connect to EKS '){
            when { expression {  params.action == 'create' } }
        steps{

            script{

                sh """
                aws configure set aws_access_key_id "$ACCESS_KEY"
                aws configure set aws_secret_access_key "$SECRET_KEY"
                aws configure set region "${params.Region}"
                aws eks --region ${params.Region} update-kubeconfig --name ${params.cluster}
                """
            }
        }
        } 
        stage('Deployment on EKS Cluster'){
            when { expression {  params.action == 'create' } }
            steps{
                script{
                  
                  def apply = false

                  try{
                    input message: 'please confirm to deploy on eks', ok: 'Ready to apply the config ?'
                    apply = true
                  }catch(err){
                    apply= false
                    currentBuild.result  = 'UNSTABLE'
                  }
                  if(apply){

                    sh """
                      kubectl apply -f .
                    """
                  }
                }
            }
        }*/  
    }
}
       