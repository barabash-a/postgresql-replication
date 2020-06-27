pipeline {
  agent {
   docker {
      image 'docker:dind-rootless'
      args '-v /var/run/docker.sock:/var/run/docker.sock''
    }

  }
  stages {
    stage('Build') {
      steps {
        sh 'docker network create mynet'
        sh 'docker build -t replication/psql .'
      }
    }

    stage('MasterDB') {
      steps {
        sh 'docker run --name master-db -d -p 15432:5432 --net mynet -e POSTGRES_DB=mydb -e POSTGRES_HOST_AUTH_METHOD=trust -v /$PWD/postgres:/var/lib/postgresql/data replication/psql'
      }
    }

    stage('Verify MasterDB') {
      steps {
        sleep 20
        sh 'docker logs master-db'
      }
    }

    stage('Backup') {
      steps {
        sh '''




docker exec master-db /bin/bash -c \'pg_basebackup -h master-db -U replicator -p 5432 -D /tmp/postgresslave -Fp -Xs -P -Rv\' '''
        sh 'docker cp master-db:/tmp/postgresslave /$PWD/'
      }
    }

    stage('SlaveDB') {
      steps {
        sh 'docker run --name slave-db -d -p 15433:5432 -e POSTGRES_DB=mydb -e POSTGRES_HOST_AUTH_METHOD=trust -v /$PWD/postgresslave:/var/lib/postgresql/data --net mynet replication/psql'
        sleep 2
      }
    }

    stage('Test') {
      steps {
        sleep 15
        sh 'docker exec master-db psql -U postgres -c \'select * from pg_stat_replication;\''
        echo 'Complete'
        cleanWs(cleanWhenFailure: true, cleanWhenAborted: true, cleanWhenSuccess: true)
      }
    }

  }
}
