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
                echo "Clonando el repositorio..."
                git branch: 'master', url: 'https://github.com/jllg18/End-to-End-Kubernetes-DevSecOps-Tetris-Project.git'
                stash includes: '**/*', name: 'terraform-code'
            }
        }

        stage('Setup Terraform Directory') {
            steps {
                echo "Preparando el directorio EKS-TF..."
                unstash 'terraform-code'
                dir('EKS-TF') {
                    sh 'ls -la'
                }
            }
        }

        stage('Terraform Init') {
            steps {
                echo "Inicializando Terraform..."
                withAWS(credentials: 'aws-key', region: 'us-east-1') {
                    dir('EKS-TF') {
                        sh 'terraform init'
                    }
                }
            }
        }

        stage('Terraform Validate') {
            steps {
                echo "Validando configuración de Terraform..."
                withAWS(credentials: 'aws-key', region: 'us-east-1') {
                    dir('EKS-TF') {
                        sh 'terraform validate'
                    }
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                echo "Generando plan de Terraform..."
                withAWS(credentials: 'aws-key', region: 'us-east-1') {
                    dir('EKS-TF') {
                        sh "terraform plan -var-file=${params.'File-Name'}"
                    }
                }
            }
        }

        stage('Terraform Apply/Destroy') {
            steps {
                echo "Ejecutando acción de Terraform..."
                withAWS(credentials: 'aws-key', region: 'us-east-1') {
                    dir('EKS-TF') {
                        script {
                            if (params.'Terraform-Action' == 'apply') {
                                sh "terraform apply -auto-approve -var-file=${params.'File-Name'}"
                            } else if (params.'Terraform-Action' == 'destroy') {
                                sh "terraform destroy -auto-approve -var-file=${params.'File-Name'}"
                            } else {
                                error "Acción de Terraform no válida: ${params.'Terraform-Action'}"
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
        success {
            echo "Pipeline completado exitosamente."
        }
        failure {
            echo "El pipeline falló. Revisa los logs para más detalles."
        }
    }
}
