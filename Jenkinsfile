pipeline{
    agent any
    tools {
        // Install the NodeJS version configured as "node-js-22-4-0" and add it to the path.
        nodejs "node-js-22-4-0"
    }

    stages{
        stage("Install Dependencies"){
            steps{
                sh 'npm install --no-audit'
            }
        }
        stage("NPM Dependenct Audit"){
            steps{
                sh 'npm install --audit-level=critical'
            }
        }
    }
}