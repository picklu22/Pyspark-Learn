pipeline {

    agent any

    stages {

        stage('Checkout') {

            steps {

                git branch: 'main',
                    url: 'https://github.com/your-user/my-python-project.git'
            }
        }

        stage('Install Dependencies') {

            steps {

                sh '''
                    python3 -m pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {

            steps {

                sh '''
                    pytest tests/
                '''
            }
        }

        stage('Build') {

            steps {

                sh '''
                    echo "Build successful"
                '''
            }
        }

        stage('Deploy') {

            steps {

                sh '''
                    echo "Deploying application..."
                '''
            }
        }
    }

    post {

        success {

            echo 'Pipeline completed successfully.'
        }

        failure {

            echo 'Pipeline failed.'
        }
    }
}
