pipeline {
    agent any

    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'False'
        KUBECONFIG = "/var/lib/jenkins/workspace/rke2-ansible-pipeline/kubeconfig"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git url: 'git@github.com:Vanukurilikhita/ansible-rke2.git',
                    branch: 'main',
                    credentialsId: 'github-ssh-vanukurilikhita'
            }
        }

        stage('Uninstall Old RKE2') {
            steps {
                sh 'ansible-playbook -i hosts.ini uninstall-rke2.yml || true'
            }
        }

        stage('Install Master Node') {
            steps {
                sh 'ansible-playbook -i hosts.ini install-master.yml'
            }
        }

        stage('Join Worker Node') {
            steps {
                sh 'ansible-playbook -i hosts.ini join-worker.yml'
            }
        }

        stage('Fix Kubeconfig Server IP') {
            steps {
                sh '''
                    echo "🔧 Fixing kubeconfig IP..."
                    sed -i 's/127.0.0.1/192.168.56.101/g' /var/lib/jenkins/workspace/rke2-ansible-pipeline/kubeconfig
                '''
            }
        }

        stage('Wait for Nodes to Become Ready') {
            steps {
                script {
                    echo "⏳ Waiting for master and worker to become Ready (max 2 minutes)..."

                    retry(12) {
                        sleep 10
                        sh '''
                        export KUBECONFIG=/var/lib/jenkins/workspace/rke2-ansible-pipeline/kubeconfig
                        READY_COUNT=$(kubectl get nodes --no-headers | grep -c " Ready")
                        if [ "$READY_COUNT" -lt 2 ]; then
                            echo "Nodes not ready yet..."
                            exit 1
                        fi
                        '''
                    }
                }
            }
        }

        stage('Show Final Cluster Status') {
            steps {
                sh '''
                export KUBECONFIG=/var/lib/jenkins/workspace/rke2-ansible-pipeline/kubeconfig
                echo "🎉 FINAL CLUSTER STATUS:"
                kubectl get nodes -o wide
                '''
            }
        }
    }
}
