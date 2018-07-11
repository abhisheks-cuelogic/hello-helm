//def label = "worker-${UUID.randomUUID().toString()}"

podTemplate(label: mypod, namespace: 'jenkins', serviceAccount: 'jenkins', containers: [
  containerTemplate(name: 'docker', image: 'docker:18.03', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.9.8', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'helm', image: 'lachlanevenson/k8s-helm:latest', command: 'cat', ttyEnabled: true)
],
volumes: [
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
]) 

{
  node(mypod) {
    def myRepo = checkout scm
    def gitCommit = myRepo.GIT_COMMIT
    def gitBranch = myRepo.GIT_BRANCH
    def shortGitCommit = "${gitCommit[0..10]}"
    def previousGitCommit = sh(script: "git rev-parse ${gitCommit}~", returnStdout: true)
    //def namespace = "dev-ecomm2"
 
    stage('Create Docker images') {
      container('docker') {
        withCredentials([[$class: 'UsernamePasswordMultiBinding',
          credentialsId: 'dockerhub',
          usernameVariable: 'DOCKER_HUB_USER',
          passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
          sh """
            docker login -u ${env.DOCKER_HUB_USER} -p ${env.DOCKER_HUB_PASSWORD}
            docker build -t ${env.DOCKER_HUB_USER}/rt-ec-migration-engine:${gitCommit} .
            docker push ${env.DOCKER_HUB_USER}/rt-ec-migration-engine:${gitCommit}
            """
        }
      }
    }
    /*stage('Run kubectl') {
      container('kubectl') {
        sh "kubectl get pods"
      }
    }*/
    stage('Deploy helm Chart') {
      container('helm') {
        sh """
         helm init --client-only --upgrade
         helm upgrade --install ecomm2-migration-engine ./helm/ --set image.tag=${gitCommit} --namespace=dev-ecomm2
         """
      }
    }
  }
}

