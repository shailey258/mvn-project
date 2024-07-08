
pipeline {
    agent any

    environment {
        //APPL_NAME = 'promotion'//

        ARTEFACT_NAME = "${WORKSPACE}/target/${APPL_NAME}-${BUILD_VERSION}.jar"

        SCAN_URL = ""

        DEV_REPO = 'staging-development'
        STAGING_TAG = "${APPL_NAME}-${BUILD_VERSION}"
        TAG_FILE = "${WORKSPACE}/tag.json"

        IQ_ID = 'Nexusiqid'
        REPO_ID = 'nexusrepo1'
    }

    stages {

        stage('Build') {
            steps {
                bat 'mvn -U -B -Dproject.version=${BUILD_VERSION} -Dmaven.test.failure.ignore clean package'
            }
            post {
                success {
                    echo 'Now archiving...'
                    archiveArtifacts artifacts: "**/target/*.jar"
                }
            }
        }

        stage('Nexus IQ Scan'){
            steps {
                script{         
                    try {
                        def policyEvaluation = nexusPolicyEvaluation failBuildOnNetworkError: true, iqApplication: selectedApplication("${iqAppID}"), iqScanPatterns: [[scanPattern: "**/*.${packaging}"]], iqStage: "${iqStage}", jobCredentialsId: ''
                        echo "Nexus IQ scan succeeded: ${policyEvaluation.applicationCompositionReportUrl}"
                        IQ_SCAN_URL = "${policyEvaluation.applicationCompositionReportUrl}"
                    } 
                    catch (error) {
                        def policyEvaluation = error.policyEvaluation
                        echo "Nexus IQ scan vulnerabilities detected', ${policyEvaluation.applicationCompositionReportUrl}"
                        throw error
                    }
                }
            }
        }

     //   stage('Nexus IQ Scan'){
       //     steps {
         //       script{
              
             //       try {
           //             def policyEvaluation = nexusPolicyEvaluation advancedProperties: '', 
             //                           enableDebugLogging: false, 
               //                         failBuildOnNetworkError: false, 
                 //                      iqApplication: manualApplication("$appl_name"),
                   //                     iqScanPatterns: [[scanPattern: '**/*.jar']], 
                    //                 iqInstanceId: '${IQ_ID}', 
                      //                    iqStage: 'build', 
                        //                 jobCredentialsId: 'Sonatype'
                        //echo $APPL_NAME
                        //echo $IQ_ID

                        //echo "Nexus IQ scan succeeded: ${policyEvaluation.applicationCompositionReportUrl}"
                        //SCAN_URL = "${policyEvaluation.applicationCompositionReportUrl}"
                    //} 
                    //catch (error) {
                      //  def policyEvaluation = error.policyEvaluation
                        //echo "Nexus IQ scan vulnerabilities detected', ${policyEvaluation.applicationCompositionReportUrl}"
                        //throw error
                    //}
                //}
            //}
        //}

        stage('Create tag'){
            steps {
                script {

                    // Git data (Git plugin)
                    echo "${GIT_COMMIT}"
                    echo "${GIT_URL}"
                    echo "${GIT_BRANCH}"
                    echo "${WORKSPACE}"

                    // construct the meta data (Pipeline Utility Steps plugin)
                    def tagdata = readJSON text: '{}'
                   // tagdata.buildUser = "${USER}" as String
                    tagdata.buildNumber = "${BUILD_NUMBER}" as String
                    tagdata.buildId = "${BUILD_ID}" as String
                    tagdata.buildJob = "${JOB_NAME}" as String
                    tagdata.buildTag = "${BUILD_TAG}" as String
                    tagdata.appVersion = "${BUILD_VERSION}" as String
                    tagdata.buildUrl = "${BUILD_URL}" as String
                    tagdata.iqScanUrl = "${SCAN_URL}" as String
                    tagdata.promote = false

                    writeJSON(file: "${TAG_FILE}", json: tagdata, pretty: 4)

                    sh 'cat ${TAG_FILE}'

                    createTag nexusInstanceId: "${REPO_ID}", tagAttributesPath: "${TAG_FILE}", tagName: "${STAGING_TAG}"

                    // write the tag name to the build page (Rich Text Publisher plugin)
                    rtp abortedAsStable: false,
                        failedAsStable: false,
                        parserName: 'Confluence',
                        stableText: "Nexus Repository Tag: ${STAGING_TAG}",
                        unstableAsStable: true
                }
            }
        }

        stage('Inspect files'){
            steps {
                sh 'ls -l target'
            }
        }

        stage('Upload to Nexus Repository'){
            steps {

                script {
                    nexusPublisher nexusInstanceId: 'srm',
                        nexusRepositoryId: "${DEV_REPO}",
                        packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: 'jar', filePath: "${ARTEFACT_NAME}"]],
                        mavenCoordinate: [artifactId: 'promotion', groupId: 'org.demo', packaging: 'jar', version: "${BUILD_VERSION}"]]],
                        tagName: "${STAGING_TAG}"
                }

            }
        }
    }
}

