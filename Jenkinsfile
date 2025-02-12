pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = 'AKIARYEUCL43WCN3O4JJ'
        AWS_SECRET_ACCESS_KEY = 'v3VogI6skuk1k7qrhcoLtmFgRIBLbWjLUsI/6ki9'
        AWS_REGION = 'us-east-1'  // Change to your region
        OLD_DB_INSTANCE = 'test'
        NEW_DB_INSTANCE = 'test1'
    }
    stages {
        stage('Validate AWS CLI') {
            steps {
                script {
                    sh 'aws --version'
                }
            }
        }
        stage('Modify Database Name') {
            steps {
                script {
                    sh "aws rds modify-db-instance --db-instance-identifier ${OLD_DB_INSTANCE} --new-db-instance-identifier ${NEW_DB_INSTANCE} --apply-immediately"
                }
            }
        }
        stage('Wait for Database Rename to Complete') {
            steps {
                script {
                    def renamed = false
                    while (!renamed) {
                        def result = sh(script: "aws rds describe-db-instances --db-instance-identifier ${OLD_DB_INSTANCE} || aws rds describe-db-instances --db-instance-identifier ${NEW_DB_INSTANCE}", returnStatus: true)
                        if (result == 0) {
                            renamed = true
                        } else {
                            echo "Waiting for database rename to complete..."
                            sleep 30
                        }
                    }
                }
            }
        }
        stage('Verify New Database Name') {
            steps {
                script {
                    sh "aws rds describe-db-instances --db-instance-identifier ${NEW_DB_INSTANCE}"
                }
            }
        }
    }
}
