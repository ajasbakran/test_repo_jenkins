pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo "build"
            }
        }

        stage('Manual Approval') {
            steps {
                script {
                    def call(def environment) {
          //WP-8643 - Build Triggered By (UserName)
          def specificCause = currentBuild.getBuildCauses('hudson.model.Cause$UserIdCause')
          env.TRIGGEREDBYUSERRESPONSE="${specificCause.userName}"
          env.TRIGGEREDBYUSER = TRIGGEREDBYUSERRESPONSE.substring(1, TRIGGEREDBYUSERRESPONSE.indexOf("@"))
          //env.TRIGGEREDBYUSER = TRIGGEREDBYUSERRESPONSE.replaceAll("[\\[\\](){}]","");
          println "Build Triggered by user: ${TRIGGEREDBYUSER}"

          env.LIQUIBASE_AUTO_TRIGGER = env.TRIGGEREDBYUSER.equalsIgnoreCase("jenkinsautomation.liquibaseuser") && (env.REPO_NAME in ['KomliAdServer', 'AdFlex'])? 'true':'false'
          if(env.LIQUIBASE_AUTO_TRIGGER.equalsIgnoreCase('true')){
            println "liquibase auto trigger enabled..."
            env.USER_OPTION='QA-Drop'
            return
          }

          //manual build choice
          def choiceoption=''
          def switchOption=''
          if(env.CHANGE_ID!=null){
            switchOption=env.CHANGE_TARGET+'-pr'
          }else{
            switchOption=env.BRANCH_NAME
          }
          switch(switchOption) {
            case 'master':
              if(env.ACTIVATE_SVC_ENABLED.equalsIgnoreCase('true')){
                choiceoption="Build n Code Analyis\nQA-Drop"
              }else{
                choiceoption="Build n Code Analyis\nDeploy\nProd-Release\nRollback\nQA-Drop"
              }
            break
            case 'prod-parallel':
              if(env.CANARYMODE_FOR_PRODPARALLELBRANCH.equalsIgnoreCase('true')){
                choiceoption="Build n Code Analyis\nDeploy\nProd-Release\nRollback\nQA-Drop"
              }
            break
            case 'master-pr':
            choiceoption="Build n Code Analyis\nMerge\nQA-Drop\nScan PR With GenAi"
            break
            case 'develop':
            choiceoption="Build n Code Analyis\nQA-Drop"
            break
            case 'develop-pr':
            choiceoption="Build n Code Analyis\nMerge\nQA-Drop"
            break
            default:
               	if (env.REPO_NAME.equalsIgnoreCase('adserver')) {
                    choiceoption="Build n Code Analyis\nDeploy-Prod-Test\nQA-Drop\nImpact-Analyzer-Run\nProd-Staging"
            	} 
              else if(env.ACTIVATE_SVC_ENABLED.equalsIgnoreCase('true') &&  env.BRANCH_NAME =~ /^(release|hotfix)/ ) {
                    choiceoption="Build n Code Analyis\nProd-Release\nQA-Drop"
              }
              else {
                    choiceoption="Build n Code Analyis\nQA-Drop"
                }
          }
          
      //***NORTHSTAR CHANGES - 28TH MARCH 2022 (WP-7406)***
                def USER_CHOICE= 'UserChoice'
                def NORTH_STAR = 'Do you want to enble build with NorthStar ?'
                def QACODECOVERAGE = 'QA Code Coverage Enable'
                def CREATE_SECRET_MGMT_DIR = 'Create Default Directories on Secret Management Server (Vault) on Both: Production and Non-Production'
                def REGRESSION_TEST_SUITES = 'Validate QA Automation Post Deployment'
                def SWAGGER_JSON_POST= 'Post SwaggerJson to SwaggerHub'

                def selectedInputChoice = [choice(name: USER_CHOICE, choices: "${choiceoption}", description: 'Select Option for this build?'), booleanParam(name: NORTH_STAR,defaultValue: false,description: 'NORTHSTAR flag to skip approvals from QA Team'), booleanParam(name: CREATE_SECRET_MGMT_DIR, defaultValue: false, description: CREATE_SECRET_MGMT_DIR), booleanParam(name: QACODECOVERAGE, defaultValue: false, description: 'Would you like to override and enable qaautomation code coverage?'), booleanParam(name: REGRESSION_TEST_SUITES, defaultValue: false, description: 'Feature will trigger QA Automation jobs, can be used for application sanity checks post deployment')]
                if (env.BRANCH_NAME.equalsIgnoreCase("ci") && env.SWAGGER_SUPPORT.equalsIgnoreCase("true")) {
                  selectedInputChoice.add(booleanParam(name: SWAGGER_JSON_POST,defaultValue: false,description: 'Do you want to deploy swaggerJson file to SwaggerHub? [only for ci branch]'))
                }
                env.USER_OPTION_ALL = input message: 'Build Options',
                ok: 'Proceed!',
                parameters: selectedInputChoice,
                submitter: 'authenticated'                      
                echo "o/p result ${env.USER_OPTION_ALL}"
                def userBuildChoice = "${env.USER_OPTION_ALL}"
                userOptions = userBuildChoice.replace('{','').replace('}','').split(",")
                for(userOption in userOptions) {
                    def option = userOption.split("=")[0].trim()
                    def value = userOption.split("=")[1].trim()
                    switch(option) {
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
                      case REGRESSION_TEST_SUITES: //WP-7404
                              if(env.REGRESSION_TEST_SUITES.equalsIgnoreCase("false"))
                                env.REGRESSION_TEST_SUITES = value
                              else
                                println "Validate QA Automation Post Deployment flag is already enabled in Jenkinsfile, skipping overried"
                              break
                      case QACODECOVERAGE: //CC-255
                              if(env.USER_OPTION.equalsIgnoreCase("Prod-Release")){
                                env.TRIGGER_QA_COVERAGE_CI_CLUSTER="false"
                                env.QAINT_AUTOMATION_CODE_COVERAGE="false"
                                echo "QA Code coverage flag is disabled if user_option: 'prod-release' "
                              }
                              else if(env.QA_AUTOMATION_COVERAGE.equalsIgnoreCase("false")){
                                env.TRIGGER_QA_COVERAGE_CI_CLUSTER = value
                               }
                              else{
                                println "QA Code coverage flag is already enabled in Jenkinsfile, skipping overried"
                              }
                              break
                    }
                }
                println "User Choice =  ${env.USER_OPTION}"  
                println "North Star =  ${env.NORTH_STAR}"  
                println "Create Secret Mgmt Dir  =  ${env.CREATE_SECRET_MGMT_DIR}"
                println "Validate QA Automation Post Deployment = ${env.REGRESSION_TEST_SUITES}"
                println "TRIGGER_QA_COVERAGE_CI_CLUSTER = ${env.TRIGGER_QA_COVERAGE_CI_CLUSTER}"

                if(env.USER_OPTION.equalsIgnoreCase("Merge")){
                  def skip_deploy= input(
                      message: 'Do you want to skip Build & Deployment for PR-Merge?',
                      parameters: [
                          choice(
                              name: 'Skip Build & Deployment', 
                              choices: ["Yes", "No"], 
                              description: 'skip build & deployment for pr-merge'
                          )
                      ]
                  )
                  env.USER_OPTION= skip_deploy.equalsIgnoreCase("Yes")?"Merge-Skip-Deploy":env.USER_OPTION
                }
                
		// Condition to disable code analysis if USER_OPTION is 'Deploy-Prod-Test' or 'Impact-Analyzer-Run' and REPO_NAME is 'adserver'
                if ((env.USER_OPTION == 'Deploy-Prod-Test' || env.USER_OPTION == 'Impact-Analyzer-Run' || env.USER_OPTION == 'Prod-Staging') && env.REPO_NAME == 'adserver') {
        	   env.CODESONAR_ANALYSIS='false'
                   env.SONARCUBE_ANALYSIS='false'
                } 
}
                }
            }
        }

        stage('Deploy') {
            steps {
                echo "deploy"
            }
        }
    }

    post {
        failure {
            echo 'Deployment failed'
        }
    }
}
