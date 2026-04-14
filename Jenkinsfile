cat > Jenkinsfile << 'EOF'
pipeline {
    agent any

    parameters {
        string(
            name: 'IMAGE_TAG',
            defaultValue: 'v1.0.1',
            description: 'Docker image tag to deploy (e.g. v1.0.5)'
        )
    }

    environment {
        ANSIBLE_DIR       = 'ansible'
        VAULT_CREDENTIALS = credentials('ansible-vault-password')
    }

    stages {
        stage('Validate Parameter') {
            steps {
                echo "Deploying image: ryanfletcher96/runcalc-pro:${params.IMAGE_TAG}"
            }
        }

        stage('Write Vault Password File') {
            steps {
                sh 'echo "${VAULT_CREDENTIALS}" > /tmp/vault_pass.txt'
                sh 'chmod 600 /tmp/vault_pass.txt'
            }
        }

        stage('Provision & Deploy') {
            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh """
                        ansible-playbook ${ANSIBLE_DIR}/provision.yml \
                          --vault-password-file /tmp/vault_pass.txt \
                          -e image_tag=${params.IMAGE_TAG}
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh """
                    PUBLIC_IP=\$(cat /tmp/prod_ip.txt)
                    echo "Checking http://\${PUBLIC_IP} ..."
                    sleep 15
                    curl --fail --retry 5 --retry-delay 5 http://\${PUBLIC_IP}
                    echo "Deployment verified!"
                """
            }
        }
    }

    post {
        always {
            sh 'rm -f /tmp/vault_pass.txt'
        }
    }
}
EOF