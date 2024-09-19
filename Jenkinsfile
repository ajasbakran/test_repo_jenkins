pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo "built"
            }
        }

        stage('Manual Approval') {
            steps {
                script {
                    def approvalInput = input(
                        id: 'manualApproval',
                        message: 'Do you want to proceed?',
                        submitter: 'user1,user2',
                        parameters: [
                            choice(
                                name: 'APPROVAL',
                                choices: 'Yes\nNo',
                                description: 'Approve or reject the deployment'
                            )
                        ]
                    )

                    if (approvalInput['APPROVAL'] == 'Yes') {  // Accessing choice parameter correctly
                        echo 'Proceeding with the deployment...'
                        // Add deployment steps here or move to the Deploy stage
                    } else {
                        error('Deployment rejected by the approver.')
                    }
                }
            }
        }

        stage('Deploy') {
            when {
                expression { approvalInput?.APPROVAL == 'Yes' }  // Ensures this stage runs only if approved
            }
            steps {
                echo "deployed"
            }
        }
    }

    post {
        failure {
            echo 'Deployment failed'
        }
    }
}
