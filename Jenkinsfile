pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID = credentials('terraform')  // Fetch AWS_ACCESS_KEY_ID from the 'terraform' credential
        AWS_SECRET_ACCESS_KEY = credentials('terraform')  // Fetch AWS_SECRET_ACCESS_KEY from the 'terraform' credential
    }

    tools {
        terraform 'terraform'  // Usa la herramienta Terraform configurada en Jenkins
    }

    stages {
        stage('Install Git') {
            steps {
                script {
                    // Install Git if it is not already installed
                    sh '''
                    if ! git --version > /dev/null 2>&1; then
                        echo "Git is not installed. Installing Git..."
                        yum update -y || true  # Ensure the script does not exit on failure
                        yum install git -y || true  # Install Git if not present
                    else
                        echo "Git is already installed."
                    fi
                    '''
                }
            }
        }

        stage('Checkout Code') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'githubToken', variable: 'GITHUB_TOKEN')]) {
                        // Verifica si el directorio ya existe
                        def repoDir = 'ADO_INFRA'
                        if (fileExists(repoDir)) {
                            echo "El repositorio ya está clonado en el directorio '${repoDir}'. Saltando clonación..."
                        } else {
                            echo "Clonando el repositorio en '${repoDir}'..."
                            sh 'git clone https://${GITHUB_TOKEN}@github.com/jllg18/ADO_INFRA.git'
                        }
                    }
                }
            }
        }

        stage('Terraform Init') {
            steps {
                dir('Transversales/ec2') {  // Navigate to the directory containing Terraform files
                    sh 'terraform init'  // Initialize Terraform configuration
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                dir('Transversales/ec2') {
                    sh 'terraform plan -out=tfplan'  // Generate and output the Terraform plan to a file
                }
            }
        }

        stage('User Decision: Apply or Destroy') {
            steps {
                script {
                    // Pregunta al usuario si desea aplicar o destruir la infraestructura
                    def userInput = input(
                        id: 'UserInput', message: '¿Qué quieres hacer después del plan?', parameters: [
                            choice(name: 'ACTION', choices: 'apply\ndestroy', description: 'Selecciona la acción a realizar:')
                        ]
                    )

                    if (userInput == 'apply') {
                        // Ejecutar terraform apply
                        dir('Transversales/ec2') {
                            sh 'terraform apply -auto-approve tfplan'  // Apply the Terraform plan
                        }
                    } else if (userInput == 'destroy') {
                        // Ejecutar terraform destroy
                        dir('Transversales/ec2') {
                            sh 'terraform destroy -auto-approve'  // Destroy the Terraform infrastructure
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                echo 'Cleaning up workspace...'
                deleteDir()  // Clean up the workspace after the job
            }
        }
        success {
            echo 'Pipeline completed successfully. Action completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}
