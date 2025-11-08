pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key')
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    credentialsId: 'github_token_id',
                    url: 'https://github.com/puneethnani56/terraaform_ansible_integration.git'
            }
        }

        stage('Terraform Init') {
            steps {
                dir('terraform') {
                    sh 'terraform init'
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                dir('terraform') {
                    sh 'terraform apply -auto-approve'
                }
            }
        }

        stage('Get Terraform Output & Generate Inventory') {
            steps {
                dir('terraform') {
                    script {
                        // Get EC2 public IP from Terraform output
                        def ip = sh(script: "terraform output -raw public_ip", returnStdout: true).trim()
                        echo "Public IP is: ${ip}"

                        // Use the secret PEM file from Jenkins
                        withCredentials([file(credentialsId: 'aws-key', variable: 'PEM_FILE')]) {
                            writeFile file: "../ansible/inventory", 
                                text: "[webserver]\n${ip} ansible_user=ubuntu ansible_ssh_private_key_file=${PEM_FILE}\n"
                        }
                    }
                }
            }
        }

        stage('Run Ansible') {
            steps {
                dir('ansible') {
                    withCredentials([file(credentialsId: 'aws-key', variable: 'PEM_FILE')]) {
                        sh '''
                        export ANSIBLE_HOST_KEY_CHECKING=False
                        ansible-playbook -i inventory playbook.yml
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}
