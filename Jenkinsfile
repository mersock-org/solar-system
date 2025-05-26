pipeline{
    agent any
    tools {
        // Install the NodeJS version configured as "node-js-22-4-0" and add it to the path.
        nodejs "node-js-22-4-0"
    }
    environment {
        MONGO_URI = "mongodb://mongo:27017/superData"
        MONGO_DB_CRED = credentials('mongo-db-cred')
        MONGO_USERNAME = credentials('mongo-db-username')
        MONGO_PASSWORD = credentials('mongo-db-password')
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
        stage("Unit Testing"){
            options { retry(2) }
            steps{
                catchError(message: 'Oops! It will be  fixed in future release', stageResult: 'UNSTABLE') {
                    sh 'npm test'
                }
            }
        }
        stage("Code coverage"){
            options { retry(2) }
            steps{
                catchError(message: 'Oops! It will be  fixed in future release', stageResult: 'UNSTABLE') {
                    sh 'npm run coverage'
                }
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
                            -Dsonar.sources=app.js \
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
                withDockerServer([uri: 'unix:///var/run/docker.sock']) {
                    sh 'docker build -t mersock/solar-system:$GIT_COMMIT .'
                }
            }
        }
    }
    post {
        always {
            junit allowEmptyResults: true, keepProperties: true, testResults: 'test-result.xml'                        
            junit allowEmptyResults: true, keepProperties: true, testResults: 'dependency-check-junit.xml'       
            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'dependency-check-report.html', reportName: 'Dependecy check report', reportTitles: '', useWrapperFileDirectly: true])                 
            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: 'coverage/lcov-report', reportFiles: 'index.html', reportName: 'Code coverrage HTML report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}
