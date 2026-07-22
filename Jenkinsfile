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
                    python3 -m pytest test/ -v \
                    --junitxml=reports/results.xml \
                    --html=reports/report.html \
                    --self-contained-html
                '''
            }
        }

        stage('Push Reports to GitHub') {
            steps {
                // Securely binds your secret token to the $GIT_TOKEN shell variable
                withCredentials([string(credentialsId: 'GITHUBTOKEN', variable: 'GIT_TOKEN')]) {
                    sh '''
                        # 1. Configure local Git user identity
                        git config user.name "Jenkins CI"
                        git config user.email "jenkins@yourdomain.com"

                        # 2. Sync your Git tracking metadata without touching workspace files
                        git fetch origin main

                        # 3. Align your local main branch pointer to match GitHub exactly
                        git checkout -B main origin/main

                        # 4. Stage your newly generated test reports directory
                        git add reports/
                        
                        # 5. Commit changes safely (keeps the build green if files match exactly)
                        git commit -m "chore: update test reports [skip ci]" --allow-empty

                        # 6. Push the committed updates directly to your remote repository
                        git push https://picklu22:${GIT_TOKEN}@//github.com main
                    '''
                }
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
