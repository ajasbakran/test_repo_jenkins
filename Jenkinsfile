pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                // Your build steps here
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

                    if (approvalInput.APPROVAL == 'Yes') { // Access the APPROVAL parameter from the map
                        echo 'Proceeding with the deployment...'
                        // Add deployment steps here
                    } else {
                        error('Deployment rejected by the approver.')
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                // Your deployment steps here
            }
        }
    }

    post {
        failure {
            echo 'Deployment failed'
        }
    }
}
