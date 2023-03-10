// week5 example uses Jenkin's "scripted" syntax, as opposed to its "declarative" syntax
// see: https://www.jenkins.io/doc/book/pipeline/syntax/#scripted-pipeline

// Defines a Kubernetes pod template that can be used to create nodes.

podTemplate(containers: [
    containerTemplate(
        name: 'gradle', image: 'gradle:6.3-jdk14', command: 'sleep', args: '30d'
        ),
    ], podRetention: onFailure()
    ) {

    node(POD_LABEL) {
        stage('Run pipeline against a gradle project - test MAIN') {
            // "container" Selects a container of the agent pod so that all shell steps are
            // executed in that container.
            git branch: 'main', url: 'https://github.com/jstmarie/week6.git'
            container('gradle') {  
                stage('Build a gradle project') {
                    if (env.BRANCH_NAME == "feature")
                      echo "Unit test on the ${env.BRANCH_NAME} branch"
                      sh '''
                      chmod +x gradlew
                      ./gradlew test
                      '''
                    }
                }

                stage("Checkstyle") {
                    if (env.BRANCH_NAME == "feature") {
                        echo "Checkstyle on the ${env.BRANCH_NAME} branch"
                        try {
                        sh '''
                        pwd
                        ./gradlew checkstyleMain
                        '''
                    } catch (Exception E) {
                        echo 'Failure detected'
                    }
                    
                    publishHTML (target: [
                      reportDir: 'build/reports/checkstyle',
                      reportFiles: 'index.html',
                      reportName: "JaCoCo checkstyle"
                    ])
                    }
                }
                
                stage("Code coverage") {
                    if (env.BRANCH_NAME == "main") {
                        echo "Code coverage on the ${env.BRANCH_NAME} branch"

                        try {
                            sh '''
                            pwd
                            ./gradlew jacocoTestCoverageVerification
                            ./gradlew jacocoTestReport
                            '''
                        } catch (Exception E) {
                            echo 'Failure detected'
                        }
                    
                        // from the HTML publisher plugin
                        // https://www.jenkins.io/doc/pipeline/steps/htmlpublisher/
                        publishHTML (target: [
                          reportDir: 'build/reports/jacoco/test/html',
                          reportFiles: 'index.html',
                          reportName: "JaCoCo Report"
                        ])
                    }
                }
            }
        }
    }
