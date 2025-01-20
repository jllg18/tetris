pipeline {
    agent {
        kubernetes {
            label 'kube-agent'
            defaultContainer 'jnlp'
            yaml """
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: jnlp
                image: jenkins/inbound-agent:4.10-3
                tty: true
            """
        }
    }
    
    parameters {
        string(name: 'File-Name', defaultValue: 'terraform.tfvars', description: 'Archivo de variables para Terraform')
        choice(name: 'Terraform-Action', choices: ['apply', 'destroy'], description: 'Acción a ejecutar en Terraform')
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/jllg18/End-to-End-Kubernetes-DevSecOps-Tetris-Project.git'
                stash includes: '**/*', name: 'terraform-code'
            }
        }
        stage('Initializing Terraform') {
            steps {
                withAWS(credentials: 'aws-key', region: 'us-east-1') {
                    dir('EKS-TF') {
                        sh 'terraform init'
                    }
                }
            }
        }
        stage('Validate Terraform Code') {
            steps {
                withAWS(credentials: 'aws-key', region: 'us-east-1') {
                    dir('EKS-TF') {
                        sh 'terraform validate'
                    }
                }
            }
        }
        stage('Terraform Plan') {
            steps {
                withAWS(credentials: 'aws-key', region: 'us-east-1') {
                    dir('EKS-TF') {
                        sh "terraform plan -var-file=${params.'File-Name'}"
                    }
                }
            }
        }
        stage('Terraform Action') {
            steps {
                withAWS(credentials: 'aws-key', region: 'us-east-1') {
                    dir('EKS-TF') {
                        script {
                            if (params.'Terraform-Action' == 'apply') {
                                sh "terraform apply -auto-approve -var-file=${params.'File-Name'}"
                            } else if (params.'Terraform-Action' == 'destroy') {
                                sh "terraform destroy -auto-approve -var-file=${params.'File-Name'}"
                            } else {
                                error "Invalid value for Terraform-Action: ${params.'Terraform-Action'}"
                            }
                        }
                    }
                }
            }
        }
    }
    
    options {
        preserveStashes()
        timestamps()
    }
    
    post {
        failure {
            echo "Pipeline failed. Check the logs for details."
        }
    }
}
