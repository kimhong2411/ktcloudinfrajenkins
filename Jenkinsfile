pipeline {
  // 실제 Jenkins 노드 label이 다르면 변경
  agent any

  environment {
    IMAGE_NAME   = 'kimhong2411/ktcloudinfra4'
    IMAGE_TAG    = '0727'
    ANSIBLE_HOST = 'master'
    DEPLOY_FILE  = '/root/deploy.yml'
  }

  stages {
    stage('Checkout') {
      steps {
        git url: 'https://github.com/kimhong2411/ktcloudinfrajenkins.git',
            branch: 'main'
      }
    }

    stage('Build and Push Image') {
      steps {
        script {
          docker.withRegistry(
            'https://index.docker.io/v1/',
            'dockerhub-credentials'
          ) {
            def builtImage =
              docker.build("${env.IMAGE_NAME}:${env.IMAGE_TAG}")

            builtImage.push()
          }
        }
      }
    }

    stage('Copy deploy.yml to Master') {
      steps {
        sh '''
          ansible "${ANSIBLE_HOST}" \
            -b \
            -m ansible.builtin.copy \
            -a "src=${WORKSPACE}/deploy.yml dest=${DEPLOY_FILE} owner=root group=root mode=0644"
        '''
      }
    }

    stage('Deploy on Kubernetes Master') {
      steps {
        sh '''
          ansible "${ANSIBLE_HOST}" \
            -b \
            -m ansible.builtin.command \
            -a "kubectl --kubeconfig=/etc/kubernetes/admin.conf apply -f ${DEPLOY_FILE}"
        '''
      }
    }

    stage('Verify Deployment') {
      steps {
        sh '''
          ansible "${ANSIBLE_HOST}" \
            -b \
            -m ansible.builtin.command \
            -a "kubectl --kubeconfig=/etc/kubernetes/admin.conf get deployment,pod,service -l app=web"
        '''
      }
    }
  }
}
