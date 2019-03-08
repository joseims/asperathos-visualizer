pipeline {
  agent any
  stages {
    stage('Unit') {
      agent any
      steps {
        sh 'tox'
      }
    }
    stage('Integration') {
      agent any
      steps {
        sh 'docker network create --attachable network-visualizer-$BUILD_ID'
        sh 'docker run -t -d --privileged --network=network-visualizer-$BUILD_ID -v /.kube:/.kube/ --name docker-visualizer-$BUILD_ID asperathos-docker'
        sh 'docker create --network=network-visualizer-$BUILD_ID --name integration-tests-visualizer-$BUILD_ID -e DOCKER_HOST=tcp://$(docker ps -aqf "name=docker-visualizer-$BUILD_ID"):2375 -e DOCKER_HOST_URL=$(docker ps -aqf "name=docker-visualizer-$BUILD_ID") integration-tests'
        sh 'docker cp . integration-tests-visualizer-$BUILD_ID:/integration-tests/test_env/visualizer/asperathos-visualizer/'
        sh 'docker start -i integration-tests-visualizer-$BUILD_ID'
      }
    }
  }
  post {
    cleanup {
      sh 'docker stop docker-visualizer-$BUILD_ID'
      sh 'docker rm -v docker-visualizer-$BUILD_ID'
      sh 'docker rm -v integration-tests-visualizer-$BUILD_ID'
      sh 'docker network rm network-visualizer-$BUILD_ID'
    }
  }
}
