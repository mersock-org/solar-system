pipeline{
    agent any
    tools {
        // Install the NodeJS version configured as "node-js-22-4-0" and add it to the path.
        nodejs "node-js-22-4-0"
    }
    environment {
    MONGO_URI = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
    }
    stages{
        stage("Install Dependencies"){
            steps{
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
                            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'dependency-check-jenkins.html', reportName: 'Dependecy check report', reportTitles: '', useWrapperFileDirectly: true])
                    }
                }
            }
        }
        stage("Unit Testing"){
            steps{
                withCredentials([usernamePassword(credentialsId: 'mongo-db-cred', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {
                    sh 'npm test'
                }
                junit allowEmptyResults: true, keepProperties: true, testResults: 'test-result.xml'                        
            }
        }
    }
}
