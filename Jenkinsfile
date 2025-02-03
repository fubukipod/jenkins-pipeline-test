pipeline {
    agent any
    
    stages {
        stage('Install Apache on Remote VM') {
            steps {
                sshagent(credentials: ['apache-server-v2']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no azureuser@20.0.161.111 \\
                        'sudo apt-get update && sudo apt-get install -y apache2'
                    """
                }
            }
        }
        
        stage('Check Apache Logs') {
            steps {                
                sshagent(credentials: ['my-ssh-creds']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no azureuser@<TARGET_SERVER_IP> \\
                        "sudo tail -n 200 /var/log/apache2/access.log | grep -E ' [45][0-9]{2} ' || echo 'No 4xx/5xx found in last 200 lines.'"
                    """
                }
            }
        }
    }
}
