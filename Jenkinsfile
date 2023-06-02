pipeline {
    agent any

     environment {
            DOCKER_TOKEN = credentials('docker-push-secret')
            DOCKER_USER = 'jgenc'
            DOCKER_SERVER = 'ghcr.io'
            DOCKER_PREFIX = 'ghcr.io/jgenc/django'
        }


    stages {
        stage('Build') {
          steps {
            sh '''
              cd myproject
              cp myproject/.env.example myproject/.env
              docker run --env-file myproject/.env $DOCKER_PREFIX:latest python manage.py test
            '''
          }
        }
        
        stage('Test') {
          steps {
            sh '''
              cd myproject
              cp myproject/.env.example myproject/.env
              docker run --env-file myproject/.env $DOCKER_PREFIX:latest python manage.py test
            '''
          }
        }

        stage("Docker Push") {
          steps {
            sh '''
              echo $DOCKER_TOKEN | docker login $DOCKER_SERVER -u $DOCKER_USER --password-stdin
              docker push $DOCKER_PREFIX --all-tags
            '''
          }
        }

        stage('Deploy db to k8s using Helm') {
            steps {
              sh '''
                helm repo add bitnami https://charts.bitnami.com/bitnami
                helm repo update
                helm upgrade --install my-db bitnami/postgresql -f k8s/db-values.yaml
              '''
            }
        }

        stage('Deploy to k8s') {
            steps {
                sh '''
                    HEAD_COMMIT=$(git rev-parse --short HEAD)
                    TAG=v0.2.3-nonroot
                    kubectl config use-context microk8s
                    kubectl apply -f k8s/django-test/django-pvc.yaml
                    kubectl apply -f k8s/django-test/django-pvc-static.yaml
                    kubectl apply -f k8s/django-test/django-deploy.yaml
                    kubectl apply -f k8s/django-test/django-service.yaml
                    kubectl set image deployment/django-app django=$DOCKER_PREFIX:$TAG
                    kubectl set image deployment/django-app djago-init=$DOCKER_PREFIX:$TAG
                '''
            }
        }

    }
}
