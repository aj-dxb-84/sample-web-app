pipeline {
  environment {
    IMAGE_REPOSITORY = "abdelghanihub/sample-web-app"
    GIT_REPOSITORY = "https://github.com/aj-dxb-84/sample-web-app.git"
    APPLICATION_NAME = "sample-web-app"
    APPLICATION_HOST = "sampleapp.somecluster.com"
  }
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: kubectl
            image: dtzar/helm-kubectl:latest
            command:
            - cat
            tty: true
          - name: docker
            image: docker:latest
            command:
            - cat
            tty: true
            volumeMounts:
             - mountPath: /var/run/docker.sock
               name: docker-sock
          volumes:
          - name: docker-sock
            hostPath:
              path: /var/run/docker.sock
        '''
    }
  }
  stages {

    stage('Build-Docker-Image') {
      steps {
        container('docker') {
          sh 'docker build -t $IMAGE_REPOSITORY:$BUILD_NUMBER .'
        }
      }
    }

    stage('Login-Into-Docker') {
      steps {
        container('docker') {
          script {
            withCredentials([
              usernamePassword(credentialsId: 'docker_hub',
                usernameVariable: 'username',
                passwordVariable: 'password')
            ]) {
              sh 'docker login -u $username -p $password'
            }
          }
        }
      }
    }

    stage('Push-Images-Docker-to-DockerHub') {
      steps {
        container('docker') {
          sh 'docker push $IMAGE_REPOSITORY:$BUILD_NUMBER'
        }
      }
    }

    stage('Deploy-To-Kubernetes') {
      steps {
        container('kubectl') {
          script {
            withCredentials([
              file(credentialsId: 'kube_config_file',
                variable: 'config')
            ]) {
              sh 'mkdir /root/.kube'
              sh 'cat $config > /root/.kube/config'
              sh 'chmod 600 /root/.kube/config'
              sh 'envsubst < manifest/kube.yaml | kubectl -n jenkins apply -f -'
            }
          }
        }
      }
    }   
  }
    post {
      always {
        container('docker') {
          sh 'docker logout'
        }
      }
    }
}
