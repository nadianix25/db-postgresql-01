pipeline {
    agent any

    parameters {
        string(name: 'REPO_URL', defaultValue: 'https://github.com/nadianix25/db-postgresql-01.git', description: 'Repository URL')
        string(name: 'BRANCH', defaultValue: 'master', description: 'Branch to checkout')
    }

    environment {
        // Replace with your database connection details
        DB_URL = 'jdbc:postgresql://db-postgresql-01.c4u0iygcr8b5.us-east-1.rds.amazonaws.com'
        DB_USER = credentials('db-username')  // Jenkins credentials ID for the database username
        DB_PASSWORD = credentials('db-password')  // Jenkins credentials ID for the database password
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
                curl -L https://repo1.maven.org/maven2/org/flywaydb/flyway-commandline/11.1.0/flyway-commandline-11.1.0-linux-x64.tar.gz | tar xz
                sudo ln -s $(pwd)/flyway-11.1.0/flyway /usr/local/bin/flyway
                '''
            }
        }

        stage('Validate Migrations') {
            steps {
                echo 'Validating migrations...'
                sh '''
                flyway -url=$DB_URL -user=$DB_USER -password=$DB_PASSWORD \
                    -locations=filesystem:sql/migrations validate
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
