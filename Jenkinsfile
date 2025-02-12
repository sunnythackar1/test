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

        stage('Create Snapshot') {
            steps {
                script {
                    echo "Creating snapshot $SNAPSHOT_NAME for DB instance $DB_INSTANCE"
                    sh """
                        aws rds create-db-snapshot --db-instance-identifier $DB_INSTANCE \
                        --db-snapshot-identifier $SNAPSHOT_NAME
                    """
                }
            }
        }

        stage('Wait for Snapshot Completion') {
            steps {
                script {
                    echo "Waiting for snapshot to be available..."
                    sh """
                        aws rds wait db-snapshot-available --db-snapshot-identifier $SNAPSHOT_NAME
                    """
                }
            }
        }

        stage('Create New Database from Snapshot') {
            steps {
                script {
                    echo "Creating new database instance $NEW_DB_INSTANCE from snapshot $SNAPSHOT_NAME"
                    sh """
                        aws rds restore-db-instance-from-db-snapshot \
                        --db-instance-identifier $NEW_DB_INSTANCE \
                        --db-snapshot-identifier $SNAPSHOT_NAME
                    """
                }
            }
        }

        stage('Wait for New Database to be Available') {
            steps {
                script {
                    echo "Waiting for the new database instance $NEW_DB_INSTANCE to be available..."
                    sh """
                        aws rds wait db-instance-available --db-instance-identifier $NEW_DB_INSTANCE
                    """
                }
            }
        }

        stage('Delete Old Database (Optional)') {
            steps {
                input message: "Delete old database $DB_INSTANCE?", ok: "Proceed"
                script {
                    echo "Deleting old database $DB_INSTANCE"
                    sh """
                        aws rds delete-db-instance --db-instance-identifier $DB_INSTANCE --skip-final-snapshot
                    """
                }
            }
        }
    }
}
