pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = 'AKIARYEUCL43WCN3O4JJ'
        AWS_SECRET_ACCESS_KEY = 'v3VogI6skuk1k7qrhcoLtmFgRIBLbWjLUsI/6ki9'
        AWS_REGION = 'us-east-1'  // Change to your region
        DB_INSTANCE = 'test'
        NEW_DB_INSTANCE = 'test1'
        SNAPSHOT_NAME = "test-snapshot-${UUID.randomUUID().toString()}"
    }
    stages {
        stage('Configure AWS CLI') {
            steps {
                script {
                    sh """
                        aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                        aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                        aws configure set region $AWS_REGION
                    """
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
        stage('Wait for Modification to Complete') {
            steps {
                script {
                    sh "aws rds wait db-instance-available --db-instance-identifier ${NEW_DB_INSTANCE}"
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
