pipeline {
  agent any
  environment {
    AWS_REGION = 'us-east-1'             // N. Virginia (your "1a" is an AZ in this region)
    ECR_REPO   = 'eks-hello'             // change if you like
    IMAGE_TAG  = "${env.BUILD_NUMBER}"   // unique tag per build
  }
  stages {
    stage('Prep AWS Vars') {
      steps {
        script {
          env.ACCOUNT_ID = sh(returnStdout: true, script: "aws sts get-caller-identity --query Account --output text").trim()
          env.ECR_URI = "${env.ACCOUNT_ID}.dkr.ecr.${env.AWS_REGION}.amazonaws.com/${env.ECR_REPO}"
          echo "Using ECR: ${env.ECR_URI}"
        }
      }
    }
    stage('Login to ECR') {
      steps {
        sh '''
          set -e
          aws ecr describe-repositories --repository-names ${ECR_REPO} --region ${AWS_REGION} >/dev/null 2>&1 || \
          aws ecr create-repository --repository-name ${ECR_REPO} --region ${AWS_REGION}
          aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
        '''
      }
    }
    stage('Build & Push Image') {
      steps {
        sh '''
          set -e
          docker build -t ${ECR_URI}:${IMAGE_TAG} .
          docker push ${ECR_URI}:${IMAGE_TAG}
        '''
      }
    }
    stage('Render K8s YAML') {
      steps {
        sh '''
          set -e
          mkdir -p k8s_rendered
          sed "s|REPLACE_ME_IMAGE|${ECR_URI}:${IMAGE_TAG}|g" k8s/deployment.yaml > k8s_rendered/deployment.yaml
          cp k8s/service.yaml k8s_rendered/service.yaml
        '''
        archiveArtifacts artifacts: 'k8s_rendered/*', fingerprint: true
      }
    }
    stage('Deploy to EKS') {
      steps {
        sh '''
          set -e
          # If your cluster name is different, change it here:
          aws eks update-kubeconfig --name preetha-eks --region ${AWS_REGION}
          kubectl apply -f k8s_rendered/deployment.yaml
          kubectl apply -f k8s_rendered/service.yaml
          kubectl rollout status deployment/hello-eks --timeout=180s || true
          kubectl get svc hello-eks-svc -o jsonpath='{.status.loadBalancer.ingress[0].hostname}' > elb.txt || true
        '''
        script {
          def elb = sh(returnStdout: true, script: "cat elb.txt || true").trim()
          echo "ELB (open in browser): http://${elb}"
        }
      }
    }
  }
  post {
    always {
      sh 'docker image prune -f || true'
    }
  }
}
