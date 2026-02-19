pipeline {
    agent {
        docker {
            image 'postman/newman:latest'
        }
    }

    parameters {
        choice(
            name: 'ENV_FILE',
            choices: ['API_Example_Env.postman_environment.json'],
            description: 'Select Postman Environment'
        )
    }

    stages {

        stage('Run API Test') {
            steps {
                sh """
                newman run API_Example_Collection.postman_collection.json \
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