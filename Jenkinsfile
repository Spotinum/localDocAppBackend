pipeline {
    agent any


    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'git@github.com:Spotinum/localDocWebApp.git'
            }
        }
        stage('Test') {
            steps {
                sh './mvnw test'
            }
        }
        stage('run ansible pipeline') {
            steps {
                build job: 'ansible'
            }
        }
        stage('Install postgres') {
            steps {
                sh '''
                    export ANSIBLE_CONFIG=~/workspace/ansible/ansible.cfg
                    ansible-playbook -i ~/workspace/ansible/hosts.yaml -l dbserver-vm ~/workspace/ansible/playbooks/postgres.yaml
                '''
            }
        }

        stage('Deploy spring boot app') {
            steps {
                sh '''
                   # replace dbserver in host_vars
                     sed -i 's/dbserver/localhost/g' ~/workspace/ansible/host_vars/appserver-vm.yaml
                     sed -i 's/192.168.56.101/20.0.144.33/g' ~/workspace/ansible/group_vars/appservers.yaml
                     sed -i 's/192.168.56.103/20.117.107.2/g' ~/workspace/ansible/group_vars/appservers.yaml
                    #  sed -i 's/80/8080/g' ~/workspace/ansible/files/nginx.http.j2
                '''
                sh '''
                    # edit host var for appserver

                    export ANSIBLE_CONFIG=~/workspace/ansible/ansible.cfg
                    ansible-playbook -i ~/workspace/ansible/hosts.yaml -l appserver-vm ~/workspace/ansible/playbooks/spring.yaml
                '''
            }
        }
       stage('Deploy frontend') {
            steps {
                sh '''
                   # sed -i 's/dbserver/20.0.144.33/g' ~/workspace/ansible/host_vars/appserver-vm.yaml
                    export ANSIBLE_CONFIG=~/workspace/ansible/ansible.cfg
                    ansible-playbook -i ~/workspace/ansible/hosts.yaml -l frontend-vm ~/workspace/ansible/playbooks/vuejs.yaml
                '''
            }
       }
    }

}