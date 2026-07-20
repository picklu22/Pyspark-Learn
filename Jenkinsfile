pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv venv

                    . venv/bin/activate

                    pip install -r requirements.txt

                    pip install pytest pytest-html
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    . venv/bin/activate

                    mkdir -p reports

                    pytest tests/ -v \
                    --junitxml=reports/results.xml \
                    --html=reports/report.html \
                    --self-contained-html
                '''
            }
        }
    }

    post {

        always {

            junit 'reports/results.xml'

            archiveArtifacts artifacts: 'reports/*', fingerprint: true
        }

        success {
            echo 'All tests passed successfully!'
        }

        failure {
            echo 'Some tests failed. Check the Jenkins report.'
        }
    }
}
