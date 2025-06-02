pipeline{
    agent any
    tools {
        // Install the NodeJS version configured as "node-js-22-4-0" and add it to the path.
        nodejs "node-js-22-4-0"
    }
    environment {
        SONAR_SCANNER_HOME = tool 'sonarqube-scanner-7.1.0'
    }
    options {
    disableResume()
    disableConcurrentBuilds abortPrevious: true
    }

    stages{
        stage("Install Dependencies"){
            options { timestamps() }
            steps{
                // sh 'sleep 300s'
                sh 'npm install --no-audit'
            }
        }

        stage("Scanning Dependencies"){
            parallel{
                stage("NPM Dependencie Audit"){
                    steps{
                        sh '''
                        npm audit --audit-level=critical
                        echo $?
                        '''
                    }
                }
                stage("OWASP Dependencie check"){
                    steps{
                            dependencyCheck additionalArguments: ''' 
                            -o \'./\'
                            -s \'./\'
                            -f \'ALL\' 
                            --disableYarnAudit
                            --prettyPrint''', odcInstallation: 'OWASP-DepCheck-10'

                            dependencyCheckPublisher failedTotalCritical: 1, pattern: 'dependency-check-report.xml', stopBuild: false
                    }
                }
            }
        }
        stage("Unit Testing E2E"){
            steps{
                    sh 'npm run test:e2e'
                }
        }
        stage("Code coverage"){
            steps{
                    sh 'npm run test'
            }
        }
        stage("SAST - SonarQube"){
            steps{
                timeout(time: 60, unit: 'SECONDS') {
                    withSonarQubeEnv('sonar-qube-server') {
                        sh 'echo $SONAR_SCANNER_HOME'
                        sh '''
                        $SONAR_SCANNER_HOME/bin/sonar-scanner \
                            -Dsonar.projectKey=solar-system-project \
                            -Dsonar.sources=app/server.js \
                            -Dsonar.javascript.lcov.reportPaths=./coverage/lcov.info
                    '''
                    }
                    waitForQualityGate abortPipeline: true
              }
            }
        }
        stage("Docker Build"){
            steps{
                sh 'printenv'
                sh 'docker build -t mersock/solar-system:$GIT_COMMIT .'
                sh 'docker save mersock/solar-system:$GIT_COMMIT > image.tar'
            }
        }
        stage("Trivy Vulnerability Scanner"){
            steps{
                sh '''
                    trivy -v

                    trivy image mersock/solar-system:$GIT_COMMIT \
                    --severity LOW, MEDIUM \
                    --exit-code 0 \
                    --quiet \
                    --input ./image.tar \
                    --format json -o trivy-image-MEDIUM-results.json

                    trivy image mersock/solar-system:$GIT_COMMIT\
                    --severity HIGH, CRITICAL \
                    --exit-code 1 \
                    --quiet \
                    --input ./image.tar \
                    --format json -o trivy-image-CRITICAL-results.json

                    rm -rf image.tar
                '''
            }
            // post{
            //     always{
            //         sh '''
            //             trivy convert \
            //             --format template --template "@/usr/local/share/trivy/templates/html.tpl" \
            //             --output trivy-image-MEDIUM-results.html trivy-image-MEDIUM-results.json

            //             trivy convert \
            //             --format template --template "@/usr/local/slare/trivy/templates/html.tp1" \
            //             --output trivy-image-CRITICAL-results.html trivy-image-CRITICAL-results.json

            //             trivy convert \
            //             --format template --template "@/usr/local/share/trivy/templates/junit.tpl" \
            //             --output trivy-image-MEDIUM-results.xml trivy-image-MEDIUM-results.json

            //             trivy convert \
            //             --format template --template "@/usr/local/share/trivy/templates/junit.tpl" \
            //             --output trivy-image-CRITICAL-results.xml trivy-image-CRITICAL-results.json
            //         '''
            //     }
            // }
        }
    }
    post {
        always {
            // junit allowEmptyResults: true, keepProperties: true, testResults: 'test-result.xml'                        
            junit allowEmptyResults: true, keepProperties: true, testResults: 'dependency-check-junit.xml'         
            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'dependency-check-jenkins.html', reportName: 'Dependecy check report', reportTitles: '', useWrapperFileDirectly: true])                 
            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: 'coverage/lcov-report', reportFiles: 'index.html', reportName: 'Code coverrage HTML report', reportTitles: '', useWrapperFileDirectly: true])

            // junit allowEmptyResults: true, keepProperties: true, testResults: 'trivy-image-CRITICAL-results.xml'   
            // junit allowEmptyResults: true, keepProperties: true, testResults: 'trivy-image-MEDIUM-results.xml'   
            // publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, reportDir: './', reportFiles: 'trivy-image-CRITICAL-results.html', reportName: 'Trivy Image Critical Vul Report',reportTitles: '', useWrapperFileDirectly: true])
            // publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, reportDir: './', reportFiles: 'trivy-image-MEDIUM-results.html', reportName: 'Trivy Image Medium Vul Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}
