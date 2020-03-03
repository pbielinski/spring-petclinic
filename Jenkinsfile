HOST="192.168.13.4"
PATH="/opt/xtrack/petclinic/"
USER_NAME="p.bielinski"

node {
   def mvnHome
   stage('Preparation') { // for display purposes
      git 'https://github.com/spring-projects/spring-petclinic.git'
      mvnHome = tool 'M3'
   }
   stage('Build') {
      withEnv(["MVN_HOME=$mvnHome"]) {
        sh '"$MVN_HOME/bin/mvn" -Dmaven.test.failure.ignore clean package'
      }
   }
   stage("Deploy") {
     stage("Test") {
        sshagent(['petclinic']) {
           sh("scp -oStrictHostKeyChecking=no target/*.jar '${USER_NAME}@${HOST}:${PATH}'")
           sh("scp -oStrictHostKeyChecking=no build/Dockerfile '${USER_NAME}@${HOST}:${PATH}'")
           sh("scp -oStrictHostKeyChecking=no build/docker-compose.yml '${USER_NAME}@${HOST}:${PATH}'")
           sh("ssh -oStrictHostKeyChecking=no -t ${USER_NAME}@${HOST}:${PATH} 'cd ${PATH}; docker-compose build'")
           sh("ssh -oStrictHostKeyChecking=no -t ${USER_NAME}@${HOST}:${PATH} 'cd ${PATH}; docker-compose up -d'")
        }
     }
   }
   stage('Results') {
      junit '**/target/surefire-reports/TEST-*.xml'
      archiveArtifacts 'target/*.jar,build/*'
   }
}
