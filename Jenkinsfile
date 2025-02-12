pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = 'AKIARYEUCL43WCN3O4JJ'
        AWS_SECRET_ACCESS_KEY = 'v3VogI6skuk1k7qrhcoLtmFgRIBLbWjLUsI/6ki9'
        AWS_REGION = 'us-east-1'  // Change to your region
        OLD_DB_INSTANCE = 'test'
        NEW_DB_INSTANCE = 'test1'
        RETRY_COUNT = 10  // Number of retries for waiting
        SLEEP_TIME = 30   // Time in seconds between retries
    }
    stages {
        stage('Modify DB Instance Name') {
            steps {
                script {
                    echo "Renaming RDS instance from ${OLD_DB_INSTANCE} to ${NEW_DB_INSTANCE}"
                    def modifyResult = sh(script: """
                        aws rds modify-db-instance --db-instance-identifier ${OLD_DB_INSTANCE} \
                        --new-db-instance-identifier ${NEW_DB_INSTANCE} --apply-immediately
                    """, returnStdout: true).trim()
                    echo "Modify Response: ${modifyResult}"
                }
            }
        }
        
        stage('Wait for Database Rename to Complete') {
            steps {
                script {
                    def retries = 0
                    while (retries < env.RETRY_COUNT.toInteger()) {
                        def renameStatus = sh(script: """
                            aws rds describe-db-instances --db-instance-identifier ${OLD_DB_INSTANCE} \
                            --query 'DBInstances[0].PendingModifiedValues.DBInstanceIdentifier' --output text 2>/dev/null || echo "not_found"
                        """, returnStdout: true).trim()

                        if (renameStatus == "not_found") {
                            echo "Old DB instance '${OLD_DB_INSTANCE}' is no longer available. Checking for new DB instance '${NEW_DB_INSTANCE}'..."
                            break
                        }
                        
                        echo "Rename still pending... waiting ${env.SLEEP_TIME} seconds"
                        sleep env.SLEEP_TIME.toInteger()
                        retries++
                    }
                }
            }
        }
        
        stage('Verify New Database Name') {
            steps {
                script {
                    def retries = 0
                    def dbAvailable = false

                    while (retries < env.RETRY_COUNT.toInteger()) {
                        def dbStatus = sh(script: """
                            aws rds describe-db-instances --db-instance-identifier ${NEW_DB_INSTANCE} \
                            --query 'DBInstances[0].DBInstanceStatus' --output text 2>/dev/null || echo "not_found"
                        """, returnStdout: true).trim()

                        if (dbStatus == "available") {
                            echo "New DB instance '${NEW_DB_INSTANCE}' is now available!"
                            dbAvailable = true
                            break
                        }

                        echo "Waiting for '${NEW_DB_INSTANCE}' to become available... Retrying in ${env.SLEEP_TIME} seconds (${retries + 1}/${env.RETRY_COUNT})"
                        sleep env.SLEEP_TIME.toInteger()
                        retries++
                    }

                    if (!dbAvailable) {
                        error "ERROR: '${NEW_DB_INSTANCE}' did not become available in time!"
                    }
                }
            }
        }
    }
}
