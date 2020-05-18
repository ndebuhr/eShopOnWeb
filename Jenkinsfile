node {
   stage('Preparation') {
      deleteDir()
      git '${REPO}'
      sh "git checkout ${BRANCH}"
   }
   stage('Build') {
      withEnv(["DOCKER_HOST=${DOCKERHOST}"]) {
        sh "docker build --no-cache -f src/Web/Dockerfile -t ${BUILDREGISTRY}/eshop-on-web:1.0.$BUILD_NUMBER ."
        sh 'docker build --no-cache -f scanner/Dockerfile -t sonar-scanner:latest .'
      }
   }
   stage('Scan Artifacts') {
      withEnv(["DOCKER_HOST=${DOCKERHOST}"]) {
        sh 'docker run --rm -t sonar-scanner -Dsonar.projectKey=eshop-on-web -Dsonar.host.url=${SONARHOST} -Dsonar.login=${SONARTOKEN}'
      }
   }
   stage('Push') {
      withEnv(["DOCKER_HOST=${DOCKERHOST}"]) {
        sh 'echo "${DOCKERPASSWORD}" | docker login -u ${DOCKERUSER} --password-stdin https://${BUILDREGISTRY}'
        sh 'docker push ${BUILDREGISTRY}/eshop-on-web:1.0.$BUILD_NUMBER'
      }
   }
   stage('Setup Deployment Package') {
       sh 'chmod +x ./xlw'
       sh './xlw apply -v --values BUILD_NUMBER=$BUILD_NUMBER,DEPLOYMENTREGISTRY=${DEPLOYMENTREGISTRY} --xl-deploy-url ${XLDHOST} --xl-deploy-username ${XLDUSER} --xl-deploy-password ${XLDPASSWORD} -f xl-deployment-package.yaml'
   }
   stage('Docker Cleanup') {
      withEnv(["DOCKER_HOST=${DOCKERHOST}"]) {
        sh 'docker rmi -f ${BUILDREGISTRY}/eshop-on-web:1.0.$BUILD_NUMBER'
        sh 'docker rmi -f sonar-scanner:latest'
      }
   }
}
