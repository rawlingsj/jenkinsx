
pipeline {
  agent {
    label "jenkins-maven"
  }

  stages {

    stage('Build Release') {
      steps {
        container('maven') {
          sh "mvn versions:set -DnewVersion=\$(jx-release-version)"
        }
        dir ('./helm/spring-boot-web-example') {
          container('maven') {
            // until kubernetes plugin supports init containers https://github.com/jenkinsci/kubernetes-plugin/pull/229/
            sh 'cp /root/netrc/.netrc ~/.netrc'

            sh "make tag"
          }
        }
        container('maven') {
          sh "mvn clean deploy fabric8:build fabric8:push -Ddocker.push.registry=$JENKINS_X_DOCKER_REGISTRY_SERVICE_HOST:$JENKINS_X_DOCKER_REGISTRY_SERVICE_PORT"
        }
      }
    }
    stage('Deploy Staging') {

      steps {
        dir ('./helm/spring-boot-web-example') {
          container('maven') {
            sh 'make release'
            sh 'helm install . --namespace staging --name example-release'
            sh 'exposecontroller --namespace staging --http' // until we switch to git environments where helm hooks will expose services
          }
        }
      }
    }
  }
}
