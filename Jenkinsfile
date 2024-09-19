pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                // Your build steps here
                echo 'Building...'
            }
        }

        stage('User Input') {
            steps {
                script {
                    def USER_CHOICE = 'UserChoice'
                    def NORTH_STAR = 'Do you want to enable build with NorthStar?'
                    def QACODECOVERAGE = 'QA Code Coverage Enable'
                    def CREATE_SECRET_MGMT_DIR = 'Create Default Directories on Secret Management Server (Vault) on Both: Production and Non-Production'
                    def REGRESSION_TEST_SUITES = 'Validate QA Automation Post Deployment'
                    def SWAGGER_JSON_POST = 'Post SwaggerJson to SwaggerHub'

                    def selectedInputChoice = [
                        choice(
                            name: USER_CHOICE,
                            choices: "Option1\nOption2\nOption3",
                            description: 'Select Option for this build?'
                        ),
                        booleanParam(
                            name: NORTH_STAR,
                            defaultValue: false,
                            description: 'NORTHSTAR flag to skip approvals from QA Team'
                        ),
                        booleanParam(
                            name: CREATE_SECRET_MGMT_DIR,
                            defaultValue: false,
                            description: CREATE_SECRET_MGMT_DIR
                        ),
                        booleanParam(
                            name: QACODECOVERAGE,
                            defaultValue: false,
                            description: 'Would you like to override and enable QA automation code coverage?'
                        ),
                        booleanParam(
                            name: REGRESSION_TEST_SUITES,
                            defaultValue: false,
                            description: 'Feature will trigger QA Automation jobs, can be used for application sanity checks post deployment'
                        )
                    ]

                    if (env.BRANCH_NAME.equalsIgnoreCase("ci") && env.SWAGGER_SUPPORT.equalsIgnoreCase("true")) {
                        selectedInputChoice.add(
                            booleanParam(
                                name: SWAGGER_JSON_POST,
                                defaultValue: false,
                                description: 'Do you want to deploy swaggerJson file to SwaggerHub? [only for ci branch]'
                            )
                        )
                    }

                    env.USER_OPTION_ALL = input(
                        message: 'Build Options',
                        ok: 'Proceed!',
                        parameters: selectedInputChoice,
                        submitter: 'authenticated'
                    )

                    echo "User Input: ${env.USER_OPTION_ALL}"

                    def userBuildChoice = "${env.USER_OPTION_ALL}"
                    def userOptions = userBuildChoice.replace('{', '').replace('}', '').split(",")

                    for (userOption in userOptions) {
                        def option = userOption.split("=")[0].trim()
                        def value = userOption.split("=")[1].trim()
                        switch (option) {
                            case USER_CHOICE:
                                env.USER_OPTION = value
                                break
                            case NORTH_STAR:
                                env.NORTH_STAR = value
                                break
                            case CREATE_SECRET_MGMT_DIR:
                                env.CREATE_SECRET_MGMT_DIR = value
                                break
                            case SWAGGER_JSON_POST:
                                env.SWAGGER_JSON_POST = value
                                break
                            case REGRESSION_TEST_SUITES:
                                if (env.REGRESSION_TEST_SUITES.equalsIgnoreCase("false")) {
                                    env.REGRESSION_TEST_SUITES = value
                                } else {
                                    println "Validate QA Automation Post Deployment flag is already enabled in Jenkinsfile, skipping override"
                                }
                                break
                            case QACODECOVERAGE:
                                if (env.USER_OPTION.equalsIgnoreCase("Prod-Release")) {
                                    env.TRIGGER_QA_COVERAGE_CI_CLUSTER = "false"
                                    env.QAINT_AUTOMATION_CODE_COVERAGE = "false"
                                    echo "QA Code coverage flag is disabled if user_option: 'prod-release'"
                                } else if (env.QA_AUTOMATION_COVERAGE.equalsIgnoreCase("false")) {
                                    env.TRIGGER_QA_COVERAGE_CI_CLUSTER = value
                                } else {
                                    println "QA Code coverage flag is already enabled in Jenkinsfile, skipping override"
                                }
                                break
                        }
                    }
                }
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

                    if (approvalInput.APPROVAL == 'Yes') {
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
