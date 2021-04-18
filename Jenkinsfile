pipeline {
  agent any

  stages {

    stage('Build') {
      steps {
          // Build new image
          sh "docker build -t invaleed/argocd-demo-1:${env.GIT_COMMIT} ."
          // Publish new image
          sh "docker login --username $DOCKERHUB_CREDS_USR --password $DOCKERHUB_CREDS_PSW && docker push invaleed/argocd-demo-1:${env.GIT_COMMIT}"
      }
    }

    stage('Deploy E2E') {
      steps {
          sh "rm -rf argocd-demo-deploy"  
          sh "git clone https://github.com/invaleed/argocd-demo-deploy.git"
          sh "git config --global user.email 'ramadoni.ashudi@gmail.com'"

          dir("argocd-demo-deploy") {
            sh "cd ./e2e && kustomize edit set image invaleed/argocd-demo-1:${env.GIT_COMMIT}"
            sh "git commit -am 'Publish new version' && git push https://$GIT_CREDS_USR:$GIT_CREDS_PSW@github.com/invaleed/argocd-demo-deploy.git || echo 'no changes'"
        }
      }
    }

    stage('Deploy to Prod') {
      steps {
        input message:'Approve deployment?'
          dir("argocd-demo-deploy") {
            sh "cd ./prod && kustomize edit set image invaleed/argocd-demo:${env.GIT_COMMIT}"
            sh "git commit -am 'Publish new version' && git push || echo 'no changes'"
          }
      }
    }
  }
}
