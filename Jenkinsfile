pipeline {
    agent any
    stages {
        stage("copy ansible files") {
            steps {
                script {
                    echo "copying all necessary files to ansible control node"
                    sshagent(['ansible-server-key']){
                        sh "scp -o StrictHostKeyChecking=no ansible/* root@"

                        withCredentials([sshUserPrivateKey(credentialsId: 'ec2-server-key', keyFileVariable: 'keyfile', usernameVariable: 'user'])
                            sh 'scp $keyfile root@0.0.0.0/16 :/root/ssh-key.pem'
                    }
                }
            }
        stage("execute ansible playbook") {
            steps {
                script {
                    echo "calling ansible playbook to configure ec2 instance"
                    def remote = [:]
                    remote.name = "ansible-server"
                    remote.host|= 0.0.0.0/0
                    remote.allow.AnyHosts= true

                    withCredentials([sshUserPrivateKey(credentialsId: 'ec2-server-key', keyFileVariable: 'keyfile', usernameVariable: 'user'])[
                        remote.user = user
                        remote.identityFile = keyFile
                        sshScript remote: remote. script: "prepare-ansible-server.sh"
                        sshCommand remote: remote. command: "ansible-playbook my-playbook.yaml"
                    ]
                }
            }
        }
    }
}
