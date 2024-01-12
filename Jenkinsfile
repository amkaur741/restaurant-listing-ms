pipeline {
  agent any

  environment {
    DOCKERHUB_CREDENTIALS = credentials('DOCKER_HUB_CREDENTIAL')
    VERSION = "${env.BUILD_ID}"

  }

  tools {
    maven "Maven"
  }

  stages {

    stage('Maven Build'){
        steps{
        sh 'mvn clean package  -DskipTests'
        }
    }

     stage('Run Tests') {
      steps {
        sh 'mvn test'
      }
    }

      stage('Docker Build and Push') {
      steps {
          sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
          sh 'docker build -t codedecode25/restaurant-listing-service:${VERSION} .'
          sh 'docker push codedecode25/restaurant-listing-service:${VERSION}'
      }
    } 


     stage('Cleanup Workspace') {
      steps {
        deleteDir()
       
      }
    }



    stage('Update Image Tag in GitOps') {
      steps {
         checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[ credentialsId: 'git-ssh', url: 'https://github.com/amkaur741/deployment-folder.gitt']])
        script {
       sh '''
          sed -i "s/image:.*/image: kauraman20\\/restaurant-listing-service:${VERSION}/" aws/restaurant-manifest.yml
        '''
          sh 'git checkout main'
          sh 'git add .'
          sh 'git commit -m "Update image tag"'
        sshagent(['git-ssh'])
            {
                  sh('git push')
            }
        }
      }
    }

  }

}


