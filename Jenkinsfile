pipeline {
 agent any

 environment {
   GIT_COMMIT_SHORT = sh(returnStdout: true, script: '''echo $GIT_COMMIT | head -c 7''')
 }

 stages {
   stage('Prepare .env') {
     steps {
       sh 'echo GIT_COMMIT_SHORT=$(echo $GIT_COMMIT_SHORT) > .env'
     }
   }

   stage('Build database') {
     steps {
       dir('database') {
         sh 'docker image build -f database/Dockerfile -t cilist-db:$GIT_COMMIT_SHORT'
         sh 'docker image tag cilist-db:$GIT_COMMIT_SHORT ravennaras/cilist:db-v3'
         sh 'docker image push ravennaras/cilist:db-v3'
       }
     }
   }

   stage('Build backend') {
     steps {
       dir('backend') {
         sh 'docker image build -f backend/Dockerfile -t cilist-be:$GIT_COMMIT_SHORT .'
         sh 'docker image tag cilist-be:$GIT_COMMIT_SHORT ravennaras/cilist:backend-v3'
         sh 'docker image push ravennaras/cilist:backend-v3'
       }
     }
   }

   stage('Build frontend') {
     steps {
       dir('frontend') {
         sh 'docker image build -f frontend/Dockerfile -t cilist-fe:$GIT_COMMIT_SHORT .'
         sh 'docker image tag cilist-fe:$GIT_COMMIT_SHORT ravennaras/cilist:frontend-v3'
         sh 'docker image push ravennaras/cilist:frontend-v3'
       }
     }
   }

   stage('Deploy to remote server') {
     steps {
       sshPublisher(publishers: [sshPublisherDesc(configName: 'Remote Server', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''docker compose up -d
      
       sleep 40

       docker compose restart backend''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '.env,docker-compose.yaml')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
     }
   }
 }
}
