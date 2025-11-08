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

                        // Write dynamic inventory file in ansible folder
                        writeFile file: "../ansible/inventory", 
                            text: "[web]\n${ip} ansible_user=ubuntu ansible_ssh_private_key_file=/var/lib/jenkins/.ssh/id_rsa\n"
                    }
                }
            }
        }

        stage('Run Ansible') {
            steps {
                dir('ansible') {
                    // Run Ansible playbook with SSH key
                    sh '''
                    export ANSIBLE_HOST_KEY_CHECKING=False
                    ansible-playbook -i inventory playbook.yml
                    '''
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
