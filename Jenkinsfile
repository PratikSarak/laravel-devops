pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    environment {
        VM_HOST = "192.168.7.160"
        VM_USER = "test"
        DEPLOY_DIR = "/home/test/laravel-app"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/PratikSarak/laravel-devops.git'
            }
        }

        stage('Package') {
            steps {
                sh '''
                    rm -f /tmp/laravel-app.tar.gz

                    tar czf /tmp/laravel-app.tar.gz \
                        --exclude=".git" \
                        --exclude=".env" \
                        --exclude="storage/logs/*" \
                        .
                '''
            }
        }

        stage('Deploy to VM') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'target-vm-ssh',
                        usernameVariable: 'SSH_USER',
                        passwordVariable: 'SSH_PASS'
                    )
                ]) {
                    sh '''
                        sshpass -p "$SSH_PASS" ssh \
                            -o StrictHostKeyChecking=no \
                            "$SSH_USER@$VM_HOST" \
                            "mkdir -p $DEPLOY_DIR"

                        sshpass -p "$SSH_PASS" scp \
                            -o StrictHostKeyChecking=no \
                            /tmp/laravel-app.tar.gz \
                            "$SSH_USER@$VM_HOST:$DEPLOY_DIR/"

                        sshpass -p "$SSH_PASS" ssh \
                            -o StrictHostKeyChecking=no \
                            "$SSH_USER@$VM_HOST" \
                            "cd $DEPLOY_DIR && \
                             tar xzf laravel-app.tar.gz && \
                             docker compose down || true && \
                             docker compose up -d --build"
                    '''
                }
            }
        }

        stage('Post Deploy') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'target-vm-ssh',
                        usernameVariable: 'SSH_USER',
                        passwordVariable: 'SSH_PASS'
                    )
                ]) {
                    sh '''
                        sshpass -p "$SSH_PASS" ssh \
                            -o StrictHostKeyChecking=no \
                            "$SSH_USER@$VM_HOST" \
                            "cd $DEPLOY_DIR && \
                             docker compose exec -T app php artisan migrate --force"
                    '''
                }
            }
        }

        stage('Verify') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'target-vm-ssh',
                        usernameVariable: 'SSH_USER',
                        passwordVariable: 'SSH_PASS'
                    )
                ]) {
                    sh '''
                        sshpass -p "$SSH_PASS" ssh \
                            -o StrictHostKeyChecking=no \
                            "$SSH_USER@$VM_HOST" \
                            "cd $DEPLOY_DIR && docker compose ps"
                    '''
                }
            }
        }
    }
}
