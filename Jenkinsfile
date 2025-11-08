pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/puneethnani56/terraaform_ansible_integration.git'
            }
        }

        stage('Terraform Init & Apply') {
            steps {
                dir('terraform') {
                    sh 'terraform init'
                    sh 'terraform apply -auto-approve'
                    sh 'terraform output -json > ../ansible/inventory.json'
                }
            }
        }

        stage('Ansible Configuration') {
            steps {
                dir('ansible') {
                    sh 'ansible-playbook -i inventory.json site.yml'
                }
            }
        }
    }
}
