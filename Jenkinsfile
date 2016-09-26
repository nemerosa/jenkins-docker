node('docker') {
   checkout scm

   stage 'Build'

   docker.build('nemerosa/jenkins-docker').inside('--volume=/var/run/docker.sock:/var/run/docker.sock') {
      // Mounts a temporary database
      sh 'docker-compose --project-name test --file docker-compose.yml up -d'
      try {
         sh '''\
            # Gets the port of the postgres container
            DB_PORT=`docker-compose --project-name test --file docker-compose.yml port db`
            echo "Postgres port = ${DB_PORT}"
            '''
      } finally {
         // Destroys the database
         sh 'docker-compose --project-name test --file docker-compose.yml down'
      }
   }
}
