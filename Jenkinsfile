pipeline {
    agent any

    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'False'
        KUBECONFIG = "/home/vboxuser/kubeconfig"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/Likhitavanukuri/ansible-rke2.git', branch: 'main'
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

        stage('Wait for Nodes to Become Ready') {
            steps {
                script {
                    echo "⏳ Waiting for master and worker to become Ready (max 2 minutes)..."

                    retry(12) {   // 12 tries × 10 sec = 2 min
                        sleep 10
                        sh '''
                        export KUBECONFIG=/home/vboxuser/kubeconfig
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
                export KUBECONFIG=/home/vboxuser/kubeconfig
                echo "🎉 FINAL CLUSTER STATUS:"
                kubectl get nodes -o wide
                '''
            }
        }
    }
}
