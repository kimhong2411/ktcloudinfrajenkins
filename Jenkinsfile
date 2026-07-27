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
        withCredentials([
          usernamePassword(
            credentialsId: 'dockerhub-credentials',
            usernameVariable: 'DOCKERHUB_USER',
            passwordVariable: 'DOCKERHUB_TOKEN'
          )
        ]) {
          sh '''
            set +x
            echo "$DOCKERHUB_TOKEN" |
              docker login -u "$DOCKERHUB_USER" --password-stdin
            set -x
    
            trap 'docker logout >/dev/null 2>&1 || true' EXIT
    
            docker build -t "${IMAGE_NAME}:${IMAGE_TAG}" .
            docker push "${IMAGE_NAME}:${IMAGE_TAG}"
          '''
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
