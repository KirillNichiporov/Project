pipeline {
      environment {
    registry = "kirill123433353463/wordpress_project"
    registryCredential = 'dockerhub'
  }
      agent any //{label 'master'}
  stages {
    stage('Cloning Git') {
      steps {
        git 'https://github.com/KirillNichiporov/Project.git'
      }
    }
    
    stage ("Lint dockerfile") {
        agent {
            docker {
                image 'hadolint/hadolint:latest-debian'
                label 'master'
                //image 'ghcr.io/hadolint/hadolint:latest-debian'
            }
        }
        steps {
            sh 'hadolint Dockerfile | tee -a hadolint_lint.txt'
        }
        post {
            always {
                archiveArtifacts 'hadolint_lint.txt'
            }
        }
    }

    stage('Building image') {
      steps{
        script {
          //dockerImage = docker.build("$registry:$BUILD_NUMBER")
          dockerImage = docker.build registry + ":latest" , "--network host ."
        }
      }
    }
    stage('Push Image to repo') {
      steps{
        script {
          docker.withRegistry( '', registryCredential ) {
            dockerImage.push()
          }
        }
      }
    }
    stage('Deploy Worpress') {
      steps{
        script {
          catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE'){
            try{
              sh """
              helm dependency update ./wordpress
              helm upgrade --install --set mariadb.enabled=false,externalDatabase.host=192.168.203.22,externalDatabase.password=wordpress,global.storageClass=nfs-client,wordpressUsername=admin,wordpressPassword=admin --debug --wait --timeout 3m --namespace=wordpress wordpress wordpress
              """
            }
            catch(err){
              sh "helm rollback wordpress --namespace=wordpress"
            }

          }
        }
      }
    }
  }
  post {
    success {
      slackSend (color: '#00FF00', message: "Deployment success: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
    }
    failure {
      slackSend (color: '#FF0000', message: "Deployment failed: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
    }
  }
}