pipeline {
    agent any

    parameters {
        choice(
            name: 'ENV_FILE',
            choices: ['API_Example_Env.postman_environment.json'],
            description: 'Select Postman Environment'
        )
    }

    stages {

        stage('Checkout') {
            steps {
                git 'https://github.com/sipahtsanan/API-automation-project-folder.git'
            }
        }

        stage('Run API Test with Newman') {
            steps {
                sh """
                docker run --rm \
                  -v \$PWD:/etc/newman \
                  postman/newman:latest \
                  run API_Example_Collection.postman_collection.json \
                  -e ${params.ENV_FILE} \
                  -r cli,html,junit \
                  --reporter-html-export API_Test_Report.html \
                  --reporter-junit-export results.xml
                """
            }
        }

        stage('Archive Report') {
            steps {
                archiveArtifacts artifacts: 'API_Test_Report.html', allowEmptyArchive: true
                junit 'results.xml'
            }
        }
    }
}