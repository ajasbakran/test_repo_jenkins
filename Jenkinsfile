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
                    approvalInput = input(  // Declare this variable at the pipeline level or globally within the script block
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

                    if (approvalInput['APPROVAL'] == 'Yes') {
                        echo 'Proceeding with the deployment...'
                    } else {
                        error('Deployment rejected by the approver.')
                    }
                }
            }
        }

        stage('Deploy') {
            when {
                expression { approvalInput['APPROVAL'] == 'Yes' }  // Use the correct way to access the choice parameter
            }
            steps {
                // Your deployment steps here
                echo 'Deploying...'
            }
        }
    }

    post {
        failure {
            echo 'Deployment failed'
        }
    }
}
