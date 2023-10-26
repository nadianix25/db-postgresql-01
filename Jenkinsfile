pipeline {
  agent any 
  environment{
      repo = 'db-01'
      github_repo = 'https://github.com/nadianix25/db-01.git'
      prodURL ='jdbc:postgresql://db01-prod.c4u0iygcr8b5.us-east-1.rds.amazonaws.com:5432/postgres'
      staURL ='jdbc:postgresql://db01-dev.c4u0iygcr8b5.us-east-1.rds.amazonaws.com:5432/postgres'
      target = "${target_env}"
  }
  stages {
    stage('Prepare') {
      steps {
        sh 'ls -a'
        sh 'docker run --rm flyway/flyway:8.5.1 version'
        sh 'git clone --single-branch --branch ${branch} ${github_repo}'
      }
    }
    stage('validate migrations') {
      steps {
        sh 'ls -a ${repo}/sql'
        sh 'echo Validate ${target}'
        script {
            if (target== 'prod')
                {
                    sh 'docker run --rm -v $WORKSPACE/${repo}/sql:/flyway/sql flyway/flyway:8.5.1 -url=${prodURL} -user=postgres -password=${dbpassword} info'
                    sh 'docker run --rm -v $WORKSPACE/${repo}/sql:/flyway/sql flyway/flyway:8.5.1 -url=${prodURL} -user=postgres -password=${dbpassword} -ignorePendingMigrations=true validate'
                }else{
                    sh 'docker run --rm -v $WORKSPACE/${repo}/sql:/flyway/sql flyway/flyway:8.5.1 -url=${staURL} -user=postgres -password=${dbpassword} info'
                    sh 'docker run --rm -v $WORKSPACE/${repo}/sql:/flyway/sql flyway/flyway:8.5.1 -url=${staURL} -user=postgres -password=${dbpassword} -ignorePendingMigrations=true validate'
                    
                }
        }
      }
    }
    stage('Migrate') {
      steps {
          script {
            sh 'echo Migrate to prod ${target}'
            if (target== 'prod')
                {
                    sh 'docker run --rm -v $WORKSPACE/${repo}/sql:/flyway/sql flyway/flyway:8.5.1 -url=${prodURL} -user=postgres -password=${dbpassword} migrate'
                }else{
                    sh 'docker run --rm -v $WORKSPACE/${repo}/sql:/flyway/sql flyway/flyway:8.5.1 -url=${staURL} -user=postgres -password=${dbpassword} migrate'
                }
        }
      }
    }
  }
}