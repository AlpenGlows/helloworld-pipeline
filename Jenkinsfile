pipeline {
  agent any
  environment {
    REGISTRY   = "192.168.64.10:5000"
    IMAGE_NAME = "helloworld"
    KUBECONFIG = "/var/lib/jenkins/.kube/config"
    NAMESPACE  = "default"
    DEPLOY     = "web"
    CONTAINER  = "web"
  }
  stages {
    stage('Checkout') { steps { checkout scm } }
    stage('Compute TAG') {
      steps {
        script {
          env.SHORT_SHA = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
          env.IMAGE_TAG = env.SHORT_SHA
          echo "Using tag: ${env.IMAGE_TAG}"
        }
      }
    }
    stage('Docker Build & Push') {
      steps {
        sh '''
          set -e
          docker build -t $REGISTRY/$IMAGE_NAME:$IMAGE_TAG .
          docker push   $REGISTRY/$IMAGE_NAME:$IMAGE_TAG
        '''
      }
    }
    stage('Bootstrap K8s (first run safe)') {
      steps {
        sh '''
          set -e
          kubectl --kubeconfig="$KUBECONFIG" -n $NAMESPACE get deploy/$DEPLOY >/dev/null 2>&1 || \
          kubectl --kubeconfig="$KUBECONFIG" -n $NAMESPACE apply -f k8s/
        '''
      }
    }
    stage('Deploy (set image)') {
      steps {
        sh '''
          set -e
          kubectl --kubeconfig="$KUBECONFIG" -n $NAMESPACE \
            set image deployment/$DEPLOY $CONTAINER=$REGISTRY/$IMAGE_NAME:$IMAGE_TAG
          kubectl --kubeconfig="$KUBECONFIG" -n $NAMESPACE \
            rollout status deployment/$DEPLOY
        '''
      }
    }
  }
  options { timestamps() }
}
