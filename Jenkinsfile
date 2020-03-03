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
   }
   stage('Build') {
      withEnv(["MVN_HOME=$mvnHome"]) {
        sh '"$MVN_HOME/bin/mvn" -Dmaven.test.failure.ignore clean package'
      }
   }
   stage("Deploy") {
     sshagent(['petclinic']) {
        sh("scp -oStrictHostKeyChecking=no target/*.jar '${env.USER_NAME}@${env.HOST}:${env.HOST_PATH}/app.jar'")
        sh("scp -oStrictHostKeyChecking=no build/Dockerfile '${env.USER_NAME}@${HOST}:${env.HOST_PATH}/Dockerfile'")
        sh("scp -oStrictHostKeyChecking=no build/docker-compose.yml '${env.USER_NAME}@${env.HOST}:${env.HOST_PATH}/docker-compose.yml'")
        sh("ssh -oStrictHostKeyChecking=no ${env.USER_NAME}@${env.HOST} 'docker-compose -f ${env.HOST_PATH}/docker-compose.yml build'")
        sh("ssh -oStrictHostKeyChecking=no ${env.USER_NAME}@${env.HOST} 'docker-compose -f ${env.HOST_PATH}/docker-compose.yml up -d'")
     }
   }
   stage('Results') {
      junit '**/target/surefire-reports/TEST-*.xml'
      archiveArtifacts 'target/*.jar,build/*'
   }
}
