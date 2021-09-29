pipeline {
    environment {
    registry = "erdemmucahitt/node-devops"
    dockerImage = ''
  }
  agent any
  tools {nodejs "node" }
  stages {
    stage('Cloning Git') {
      steps {
        git 'https://github.com/erdemmucahitt/node-devops'
      }
    }
    stage('Build') {
       steps {
         sh 'npm install'
       }
    }
    stage('Test') {
      steps {
        sh 'npm test'
      }
    }
    stage('Building image') {
      steps{
        script {
          dockerImage = docker.build registry + ":$BUILD_NUMBER"
        }
      }
    }
    stage('Deploy Image') {
      steps{
         script {
            docker.withRegistry('https://192.168.1.102:5000', '') {
            dockerImage.push()
          }
        }
      }
    }
    stage('Remove Unused docker image from local ') {
      steps{
        sh "docker rmi $registry:$BUILD_NUMBER"
      }
    }
    
    stage('Deploy to Kubernetes Cluster') {
      steps {
      ///CREATE AND APPLY THE PATCH. REMEMBER TO LOGIN ON THE CLUSTER. (-s $CLUSTER_URL --token $TOKEN_CLUSTER --insecure-skip-tls-verify)
        sh  '''
                         
        PATCH_TO_DEPLOY={\\"metadata\\":{\\"labels\\":{\\"version\\":\\"${env.BUILD_ID}\\"}},\\"spec\\":{\\"template\\":{\\"metadata\\":{\\"labels\\":{\\"version\\":\\"${env.BUILD_ID}\\"}},\\"spec\\":{\\"containers\\":[{\\"name\\":\\"$NAME_DEPLOY\\",\\"image\\":\\"my-image:${env.BUILD_ID}\\"}]}}}}
                        
        kubectl patch deployment $NAME_DEPLOY  -n $NAMESPACE -p $PATCH_TO_DEPLOY \
        -s $CLUSTER_URL --token $TOKEN_CLUSTER --insecure-skip-tls-verify
                        
        '''
                        
        }
    }
        
    
  }
}
