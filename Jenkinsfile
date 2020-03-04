HOST_PORT=''
HOST_URL=''

node {
   def mvnHome
   stage('Preparation') { // for display purposes
      checkout([
         $class: 'GitSCM',
         branches: scm.branches,
         extensions: scm.extensions + [[$class: 'CleanCheckout']],
         userRemoteConfigs: scm.userRemoteConfigs
       ])
      mvnHome = tool 'M3'
      
      def hostWithPort = env.HOST.split(':')
      if(hostWithPort.length == 1) {
         HOST_PORT = '22'
      } else {
         HOST_PORT = hostWithPort[1]
      }
      HOST_URL = hostWithPort[0]
   }
   stage('Build') {
      withEnv(["MVN_HOME=$mvnHome"]) {
        sh '"$MVN_HOME/bin/mvn" -Dmaven.test.failure.ignore clean package'
      }
   }
   stage("Deploy") {
     sshagent(['petclinic']) {
        sh("scp -oStrictHostKeyChecking=no -P ${HOST_PORT} target/*.jar '${env.USER_NAME}@${HOST_URL}:${env.HOST_PATH}/app.jar'")
        sh("scp -oStrictHostKeyChecking=no -P ${HOST_PORT} build/Dockerfile '${env.USER_NAME}@${HOST_URL}:${env.HOST_PATH}/Dockerfile'")
        sh("scp -oStrictHostKeyChecking=no -P ${HOST_PORT} build/docker-compose.yml '${env.USER_NAME}@${HOST_URL}:${env.HOST_PATH}/docker-compose.yml'")
        sh("ssh -oStrictHostKeyChecking=no ${env.USER_NAME}@${HOST_URL} -p ${HOST_PORT} 'docker-compose --project-directory ${env.HOST_PATH} -f ${env.HOST_PATH}/docker-compose.yml build'")
        sh("ssh -oStrictHostKeyChecking=no ${env.USER_NAME}@${HOST_URL} -p ${HOST_PORT} 'docker-compose --project-directory ${env.HOST_PATH} -f ${env.HOST_PATH}/docker-compose.yml up -d'")
     }
   }
   stage('Results') {
      junit '**/target/surefire-reports/TEST-*.xml'
      archiveArtifacts 'target/*.jar,build/*'
   }
}
