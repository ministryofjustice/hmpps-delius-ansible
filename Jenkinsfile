// Parameters required for job
// parameters:
//     choice:
//       name: 'environment_type' [ dev | prod ]
//       description: 'The Bastion to configure'

def prepare_env() {
    sh '''
        docker pull mojdigitalstudio/hmpps-ansible-builder:0.40.0
    '''
}

def run_ansible(environment_type) {
    sshagent (credentials: ['bastion-key']) {
        sh """
            set +x
            docker run --rm -v `pwd`:/home/tools/data \
            -v ~/.ssh:/home/tools/.ssh \
            -v $SSH_AUTH_SOCK:/ssh-agent \
            -e SSH_AUTH_SOCK=/ssh-agent \
            mojdigitalstudio/hmpps-ansible-builder:0.40.0 bash -c \"ansible-galaxy install -r requirements.yml && \
             ansible-playbook -u jenkins --ssh-extra-args='-o StrictHostKeyChecking=no' -i inventory/${environment_type} bastion.yml\"
            set -x
        """
    }
}

pipeline {

    agent { label "jenkins_agent" }

    stages {

        stage('setup') {
            steps {
                checkout scm
                prepare_env()
            }
        }

        stage('Ansible') {
            steps {
                run_ansible(environment_type)
            }
        }

    }
}
