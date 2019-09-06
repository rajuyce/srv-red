pipeline {
   agent any
   environment {
     // You must set the following environment variables
     // ORGANIZATION_NAME
     // DOCKERHUB_USERNAME  (it doesn't matter if you don't have one)
     // REPOSITORY_DOCKERHUB
     // REPOSITORY_GITHUB
     REPOSITORY_TAG="${ORGANIZATION_NAME}/${REPOSITORY_MVN}:${BUILD_ID}"
     registry = "${ORGANIZATION_NAME}/${REPOSITORY_MVN}"
     registryCredential = 'dockerhub'
     dockerImage = ''
   }

   stages {
      stage('Preparation') {
         steps {
            cleanWs()
            git credentialsId: 'GitHub', url: "https://github.com/${ORGANIZATION_NAME}/${REPOSITORY_MVN}"
         }
      }
      stage('Create Application Build') {
         steps {
            sh '''mvn install clean package'''
         }
      }
      stage('Create Docker Image') {
         steps {
           sh 'docker image build -t ${REPOSITORY_TAG} .'
         }
      }
      stage('Push Docker Image') {
  	 steps{
	    withCredentials([usernamePassword( credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
            sh "docker login -u ${USERNAME} -p ${PASSWORD}"
            sh "docker push ${REPOSITORY_TAG}"
            }      
         }
      }
      stage('Deploy Application on EKS') {
          steps {
                    sh 'envsubst < ${WORKSPACE}/eks-app.yml | kubectl apply -f -'
          }
      }
      stage('Purge Unused Docker Image') {
         steps{
            sh "docker rmi ${REPOSITORY_TAG}"
         }
      }
   }
}
