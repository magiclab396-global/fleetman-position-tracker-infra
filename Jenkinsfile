pipeline {
   agent any

   environment {
     // You must set the following environment variables
     // ORGANIZATION_NAME
     // YOUR_DOCKERHUB_USERNAME (it doesn't matter if you don't have one)

     SERVICE_NAME = "fleetman-position-tracker"
     REPOSITORY_TAG="${YOUR_DOCKERHUB_USERNAME}/${ORGANIZATION_NAME}-${SERVICE_NAME}:${BUILD_ID}"
   }

   stages {
      stage('Preparation') {
         steps {
            cleanWs()
            git credentialsId: 'GitHub', url: "https://github.com/${ORGANIZATION_NAME}/${SERVICE_NAME}"
         }
      }
      stage('Build') {
         steps {
            sh '''mvn clean package'''
         }
      }

      stage('Build and Push Image') {
         steps {
           // sh 'docker image build -t ${REPOSITORY_TAG} .'
            withDockerRegistry([ credentialsId: "DockerHub", url: "" ]){
               sh '''  
                  echo `Build image ...`
                  docker image build -t ${REPOSITORY_TAG} .
                  echo `Push image ...`
                  docker push ${REPOSITORY_TAG}
                  echo `Done pushing image ..11..`
               '''
            }
           
         }
      }

      stage('Deploy to Cluster') {
          steps {
             // withKubeConfig([credentialsId: 'K8sSaToken', serverUrl: "${K8S_API_ENDPOINT}"]){
             //    // sh 'envsubst < ${WORKSPACE}/deploy.yaml | kubectl apply -f -'
             //    //def agentLabel
             //   // if (BRANCH_NAME =~ /^(staging|master)$/)  {
             //   //     agentLabel = "prod"
             //   // } else {
             //   //     agentLabel = "master"
             //   // } 
             // }

             script{
                sh '''  
                   def text = readFile file: "kustomization.yaml"
                   text = text.replaceAll("/(?m)^newTag:.*$/", "newTag:${BUILD_ID}") 
                   export BUILD_ID=${BUILD_ID}
                   git add . -m "Update app image tag to ${BUILD_ID}"
                   git push origin master
                '''
               }
          }
      }
   }
}
