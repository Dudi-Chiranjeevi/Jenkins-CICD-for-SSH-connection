pipeline {
    agent any

    parameters {
        string(name: 'USERNAME', defaultValue: 'ubuntu', description: 'Remote username')
        string(name: 'SSH_PORT', defaultValue: '22', description: 'SSH port')
        string(name: 'PEM_FILE', defaultValue: '/var/lib/jenkins/.ssh/keypair.pem', description: 'Path to AWS private key (.pem file)')
        string(name: 'CONFIG_FILE', defaultValue: '/var/lib/jenkins/server_list.conf', description: 'Path to server list config file')
        booleanParam(name: 'CONFIGURE_SSH', defaultValue: false, description: 'Configure SSH config for passwordless login')
    }

    stages {
        stage('Validate Parameters') {
            steps {
                script {
                    if (!fileExists(params.PEM_FILE)) {
                        error "PEM file ${params.PEM_FILE} not found."
                    }

                    if (!fileExists(params.CONFIG_FILE)) {
                        error "Configuration file ${params.CONFIG_FILE} not found."
                    }
                }
            }
        }

        stage('Read Server List') {
            steps {
                script {
                    def servers = readFile(params.CONFIG_FILE).split('\n').findAll { it.trim() }
                    env.SERVER_LIST = servers.join(',')
                }
            }
        }

        stage('Generate SSH Key') {
            steps {
                script {
                    def sshKeyPath = "$HOME/.ssh/id_rsa"
                    if (!fileExists(sshKeyPath)) {
                        sh "ssh-keygen -t rsa -b 2048 -f $sshKeyPath -q -N ''"
                        sh "sudo chown ubuntu:ubuntu $sshKeyPath"
                        echo "SSH key generated and ownership changed at $sshKeyPath"
                    } else {
                        echo "SSH key already exists at $sshKeyPath"
                    }
                }
            }
        }


        stage('Distribute SSH Key') {
            steps {
                script {
                    def servers = env.SERVER_LIST.split(',')
                    servers.each { server ->
                        try {
                            echo "Adding ${server} to known hosts..."
                            sh "ssh-keyscan -p ${params.SSH_PORT} ${server} >> $HOME/.ssh/known_hosts"

                            echo "Copying SSH key to ${server}..."
                            sh "scp -i ${params.PEM_FILE} -P ${params.SSH_PORT} $HOME/.ssh/id_rsa.pub ${params.USERNAME}@${server}:/tmp/id_rsa.pub"

                            echo "Configuring SSH key on ${server}..."
                            sh "ssh -i ${params.PEM_FILE} -p ${params.SSH_PORT} ${params.USERNAME}@${server} \"mkdir -p ~/.ssh && cat /tmp/id_rsa.pub >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys && rm /tmp/id_rsa.pub\""

                            echo "SSH key successfully configured on ${server}"
                        } catch (Exception e) {
                            echo "Failed to configure SSH key on ${server}: ${e.getMessage()}"
                        }
                    }
                }
            }
        }


        stage('Configure SSH Config (Optional)') {
            when {
                expression { return params.CONFIGURE_SSH }
            }
            steps {
                script {
                    def sshConfigPath = "$HOME/.ssh/config"
                    def servers = env.SERVER_LIST.split(',')

                    servers.each { server ->
                        def hostAlias = server.replaceAll('\\.', '-')
                        sh "echo \"Host ${hostAlias}\\n    HostName ${server}\\n    User ${params.USERNAME}\\n    Port ${params.SSH_PORT}\\n    IdentityFile $HOME/.ssh/id_rsa\\n\" >> ${sshConfigPath}"
                    }

                    sh "chmod 600 ${sshConfigPath}"
                    echo "Configuration saved in ${sshConfigPath}"
                }
            }
        }
    }

    post {
        always {
            echo 'SSH setup process completed!'
        }
    }
}
