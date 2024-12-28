pipeline {
    agent any

    parameters {
        string(name: 'REPO_URL', defaultValue: 'https://github.com/nadianix25/db-postgresql-01.git', description: 'Repository URL')
        string(name: 'BRANCH', defaultValue: 'master', description: 'Branch to checkout')
    }

    environment {
        // Replace with your database connection details
        DB_URL = 'jdbc:postgresql://db-postgresql-01.c4u0iygcr8b5.us-east-1.rds.amazonaws.com/'
        DB_USER = credentials('db-username')  // Jenkins credentials ID for the database username
        DB_PASSWORD = credentials('db-password')  // Jenkins credentials ID for the database password
        FLYWAY_HOME = '/var/lib/jenkins/workspace/db01-prostres/flyway-11.1.0'
        PATH = "${FLYWAY_HOME}:${env.PATH}"
    }

    stages {


       stage('Checkout Code') {
            steps {
                echo "Cloning repository from ${params.REPO_URL} on branch ${params.BRANCH}..."
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "*/${params.BRANCH}"]],
                ])
            }
        }

stage('Install Flyway') {
    steps {
        echo 'Installing Flyway...'
        sh '''
        # Set up Flyway version
        FLYWAY_VERSION=11.1.0

        # Download Flyway
        curl -L -o flyway.tar.gz https://repo1.maven.org/maven2/org/flywaydb/flyway-commandline/$FLYWAY_VERSION/flyway-commandline-$FLYWAY_VERSION-linux-x64.tar.gz

        # Extract Flyway into workspace
        tar -xzf flyway.tar.gz -C $WORKSPACE
        
        # Use Flyway directly from workspace
        export PATH=$WORKSPACE/flyway-$FLYWAY_VERSION:$PATH

        # Clean up downloaded file
        rm -f flyway.tar.gz
        '''
    }
}

        stage('Validate Migrations') {
          steps {
             echo 'Validating migrations...'
             sh '''
              flyway -url=$DB_URL -user=$DB_USER -password=$DB_PASSWORD \
                -locations=filesystem:sql/ migrate
              '''
          }
        }

        stage('Migrate Database') {
            steps {
                echo 'Applying database migrations...'
                sh '''
                flyway -url=$DB_URL -user=$DB_USER -password=$DB_PASSWORD \
                    -locations=filesystem:sql/migrations migrate
                '''
            }
        }
    }

    post {
        success {
            echo 'Migrations applied successfully!'
        }
        failure {
            echo 'Migration failed. Please check the logs.'
        }
    }
}
