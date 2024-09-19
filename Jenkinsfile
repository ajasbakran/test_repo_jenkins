pipeline {
    agent any

    stages {
        stage('User Input') {
            steps {
                script {
                    // Define the input parameters
                    def userInput = input(
                        message: 'Please provide your input:',
                        parameters: [
                            choice(
                                name: 'DEPLOYMENT_APPROVAL',
                                choices: ['Yes', 'No'],
                                description: 'Approve the deployment?'
                            ),
                            booleanParam(
                                name: 'ENABLE_FEATURE_X',
                                defaultValue: false,
                                description: 'Enable Feature X?'
                            ),
                            booleanParam(
                                name: 'ENABLE_FEATURE_Y',
                                defaultValue: false,
                                description: 'Enable Feature Y?'
                            )
                        ]
                    )
                    
                    // Echo the input values
                    echo "Deployment Approval: ${userInput.DEPLOYMENT_APPROVAL}"
                    echo "Enable Feature X: ${userInput.ENABLE_FEATURE_X}"
                    echo "Enable Feature Y: ${userInput.ENABLE_FEATURE_Y}"
                }
            }
        }
    }

    post {
        failure {
            echo 'The pipeline failed.'
        }
        success {
            echo 'The pipeline succeeded.'
        }
    }
}
