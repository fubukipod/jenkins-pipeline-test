pipeline {
    agent any
    
    stages {
        stage('Install Apache') {
            steps {
                sshagent(credentials: ['apache-server-v2']) {
                    script {
                        echo "Installing Apache on the remote server..."
                        sh """
                            ssh -o StrictHostKeyChecking=no azureuser@20.0.161.111 \\
                            'sudo apt-get update && sudo apt-get install -y apache2'
                        """
                        echo "✅ Apache installed successfully!"
                    }
                }
            }
        }

        stage('Check Apache Default Page') {
            steps {
                sshagent(credentials: ['apache-server-v2']) {
                    script {
                        echo "Checking Apache default page..."
                        // Make an HTTP request to the remote server's localhost
                        // and capture the HTTP status code
                        def statusCode = sh(
                            script: """
                                ssh -o StrictHostKeyChecking=no azureuser@20.0.161.111 \\
                                'curl -s -o /dev/null -w "%{http_code}" http://localhost'
                            """, 
                            returnStdout: true
                        ).trim()

                        if (statusCode == '200') {
                            echo "✅ Success: Apache default page responded with HTTP 200 OK."
                        } else {
                            error "❌ ERROR: Apache default page responded with status code ${statusCode}."
                        }
                    }
                }
            }
        }

        stage('Check Apache Logs') {
            steps {
                sshagent(credentials: ['apache-server-v2']) {
                    script {
                        echo "Inspecting Apache logs for 4xx/5xx errors..."
                        // We'll grep the access.log for lines with 4xx/5xx codes
                        // If none are found, we print a friendly message.
                        sh """
                            ssh -o StrictHostKeyChecking=no azureuser@20.0.161.111 \\
                            "grep -E ' (4[0-9]{2}|5[0-9]{2}) ' /var/log/apache2/access.log || echo 'No 4xx/5xx errors found.'"
                        """
                        echo "✅ Log inspection completed."
                    }
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Build finished successfully! All stages completed without errors."
        }
        failure {
            echo "❌ Build failed. Check the console output for more details."
        }
    }
}
