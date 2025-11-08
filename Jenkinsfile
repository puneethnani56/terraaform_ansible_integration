pipeline {
    agent any

    environment {
        TF_WORKSPACE = "dev"
        TF_DIR = "./terraform"
        ANSIBLE_DIR = "./ansible"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/puneethnani56/terraaform_ansible_integration.git'
            }
        }

        stage('Terraform Init & Plan') {
            steps {
                dir("${TF_DIR}") {
                    sh 'terraform init'
                    sh 'terraform plan -out=tfplan'
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                dir("${TF_DIR}") {
                    sh 'terraform apply -auto-approve tfplan'
                }
            }
        }

        stage('Export Terraform Outputs') {
            steps {
                dir("${TF_DIR}") {
                    // Export instance IPs or other data for Ansible
                    sh 'terraform output -json > ../ansible/inventory.json'
                }
            }
        }

        stage('Ansible Configuration') {
            steps {
                dir("${ANSIBLE_DIR}") {
                    // Convert Terraform outputs to Ansible inventory
                    sh '''
                    ansible-inventory -i inventory.json --list
                    ansible-playbook -i inventory.json site.yml
                    '''
                }
            }
        }

        stage('Post-Deployment Validation') {
            steps {
                echo "Running smoke tests..."
                sh 'curl -f http://your-app-url/health || exit 1'
            }
        }
    }

    post {
        success {
            echo "✅ Deployment completed successfully."
        }
        failure {
            echo "❌ Deployment failed. Check logs for details."
        }
    }
}
