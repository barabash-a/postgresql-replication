pipeline {
  agent {
    node {
      label 'host'
    }

  }
  stages {
    stage('Build') {
      steps {
        sh 'docker network create mynet'
        sh 'echo "Created network \'mynet\'"'
        sh 'docker build -t replication/psql .'
      }
    }

  }
}