pipeline {
    agent any

    stages {

      stage('Build') {
        steps {
          git branch: 'main', url : 'git@github.com:jgenc/django3-sampe-project.git'
        }
      }

      stage("Test") {
        steps {
          sh '''
            python3 -m venv .venv
            source .venv/bin/activate
            pip install -r requirements.txt
            cd myproject
            cp myproject/.env.example myproject/.env
            python manage.py test
          '''
        }
      }

      stage("Deploy Database") {
        steps {
          sh '''
            ansible-galaxy install geerlingguy.jenkins
            ansible-galaxy install geerlingguy.postgresql
          '''

          sh '''
            ansible-playbook -i ~/workspace/ansible-project/hosts.yml -l deploy-vm-1 ~/workspace/ansible-project/playbooks/postgres.yml
          '''
        }
      }

      stage("Deploy Django") {
        steps {
          sh '''
            ansible-playbook -i ~/workspace/ansible-project/hosts.yml -l app01
          '''

          sh '''
          '''
        }
      }


  }
}
