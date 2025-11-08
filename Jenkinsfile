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

        stage('Terraform Init & Apply') {
            steps {
                dir('terraform') {
                    sh 'terraform init'
                    sh 'terraform apply -auto-approve'
                }
            }
        }

        stage('Generate Inventory & Run Ansible') {
            steps {
                dir('terraform') {
                    script {
                        // Get EC2 public IP from Terraform output
                        def ip = sh(script: "terraform output -raw public_ip", returnStdout: true).trim()
                        echo "Public IP is: ${ip}"

                        // Use the secret PEM file and run Ansible within the same block
                        withCredentials([file(credentialsId: 'aws-key', variable: 'PEM_FILE')]) {

                            // Write dynamic inventory for Ansible
                            writeFile file: "../ansible/inventory",
                                text: "[webserver]\n${ip} ansible_user=ec2-user ansible_ssh_private_key_file=${PEM_FILE}\n"

                            // Run Ansible immediately after generating inventory
                            dir('../ansible') {
                                sh '''
                                export ANSIBLE_HOST_KEY_CHECKING=False
                                ansible-playbook -i inventory playbook.yml
                                '''
                            }
                        }
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
