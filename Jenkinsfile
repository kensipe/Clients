node {
   stage('Preparation') { // for display purposes
      // Get some code from a GitHub repository
      git 'https://github.com/kensipe/Clients.git'
   }
   stage('Build') {
      sh 'docker build -t kensipe/clients:${BUILD_ID} .'
   }
   stage('Push')
   withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub',
                    usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']])
   {
      sh 'docker login --username=$USERNAME --password=$PASSWORD'
      sh 'docker push kensipe/clients:${BUILD_ID}'
   }
   stage('Prepare Scripts')
   {
      sh 'sed -i \'s/IDTAG/\'${BUILD_ID}\'/g\' deploy/pumrpclientdeploy.yaml'
      sh 'sed -i \'s/IDPRETAG/\'$((${BUILD_ID}-1))\'/g\' deploy/pumrpclientpredeploy.yaml'
      sh 'sed -i \'s/IDTAG/\'${BUILD_ID}\'/g\' deploy/updategw50.yaml'
      sh 'sed -i \'s/IDPRETAG/\'$((${BUILD_ID}-1))\'/g\' deploy/updategw50.yaml'
      sh 'sed -i \'s/IDTAG/\'${BUILD_ID}\'/g\' deploy/updategw100.yaml'
      sh 'sed -i \'s/IDPRETAG/\'$((${BUILD_ID}-1))\'/g\' deploy/updategw100.yaml'
   }
   stage('Deploy in Cluster')
   {
       sh 'curl -v -X POST --data-binary @deploy/pumrpclientdeploy.yaml -H "Content-Type: application/x-yaml" vamp.vamp.marathon.mesos:1781/api/v1/deployments'
   }
   stage('Move GW 50/50')
   {
       input 'Do you approve deployment?'
       sh 'curl -v -X PUT --data-binary @deploy/updategw50.yaml -H "Content-Type: application/x-yaml" vamp.vamp.marathon.mesos:1781/api/v1/gateways/pumrpclient'
   }
   stage('Move Full')
   {
       input 'Do you approve deployment?'
       sh 'curl -v -X PUT --data-binary @deploy/updategw100.yaml -H "Content-Type: application/x-yaml" vamp.vamp.marathon.mesos:1781/api/v1/gateways/pumrpclient'
   }
   stage('Undeploy Previous App')
   {
       sh 'curl -v -X DELETE --data-binary @deploy/pumrpclientpredeploy.yaml -H "Content-Type: application/x-yaml" vamp.vamp.marathon.mesos:1781/api/v1/deployments/pumrpclient:$((${BUILD_ID}-1))'
       sh 'curl -v -X DELETE -H "Content-Type: application/x-yaml" vamp.vamp.marathon.mesos:1781/api/v1/breeds/pumrpclient:$((${BUILD_ID}-1))'

   }
}
