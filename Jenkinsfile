pipeline{
    agent any
    tools {
        // Install the NodeJS version configured as "node-js-22-4-0" and add it to the path.
        NodeJS "node-js-22-4-0"
    }

    stages{
        stage("VM Node Version"){
            steps{
                sh '''
                    node -v
                    npm -v
                '''
            }
        }
    }
}