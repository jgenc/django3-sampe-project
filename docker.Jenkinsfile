pipeline {
    agent any

    environment {
            DOCKER_TOKEN = credentials('docker-push-secret')
            DOCKER_USER = 'jgenc'
            DOCKER_SERVER = 'ghcr.io'
            DOCKER_PREFIX = 'ghcr.io/jgenc/django3-sampe-project'
    }

    stages {
      stage("Clone") {
        steps {
          git branch: 'main', url : 'git@github.com:jgenc/django3-sampe-project.git'
        }
      }

      stage("Docker Build") {
        steps {
          sh '''
            HEAD_COMMIT=$(git rev-parse --short HEAD) 
            TAG=$HEAD_COMMIT-$BUILD_ID
            docker build --rm -t $DOCKER_PREFIX:$TAG -t $DOCKER_PREFIX:latest -f nonroot-alpine.Dockerfile .
          '''
        }
      }
      
      stage("Test Docker container") {
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

      stage("Deploy db to k8s using Helm") {
        steps {
          sh '''
            helm repo add bitnami https://charts.bitnami.com/bitnami
            helm repo update
            helm upgrade --install my-db bitnami/postgresql -f k8s/db-helm/values.yaml
          '''
        }
      }



  }
}
