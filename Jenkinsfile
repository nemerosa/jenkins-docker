node('docker') {
   checkout scm

   stage 'Build'

   sh '''\
      HOSTIP=`ip -4 addr show scope global dev eth0 | grep inet | awk '{print $2}' | cut -d / -f 1`
      echo HOSTIP=${HOSTIP}
      echo HOSTIP=${HOSTIP} > host.properties
      '''

   String hostIP = readProperties('host.properties').getProperty('HOSTIP')
   echo "Host IP = ${hostIP}"

   docker.build('nemerosa/jenkins-docker').inside("--net host --volume=/var/run/docker.sock:/var/run/docker.sock --add-host dockerhost:${hostIP}") {
      // Mounts a temporary database
      sh 'docker-compose --project-name test --file docker-compose.yml up -d'
      try {
         sh '''\
            # Gets the port of the postgres container
            DB_PORT=`docker-compose --project-name test --file docker-compose.yml port db 5432 | awk -F ':' '{print $NF}'`
            echo "Postgres port = ${DB_PORT}"
            # Sleep a bit (giving time to the database to init)
            sleep 30s
            # Connecting to the database and creating a table
            export PGPASSWORD=ontrack
            psql -h dockerhost -p ${DB_PORT} -U ontrack ontrack << EOF
create table test ();
EOF
            '''
      } finally {
         // Destroys the database
         sh 'docker-compose --project-name test --file docker-compose.yml down'
      }
   }
}
