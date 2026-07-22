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
                // Ensure you have added a 'Username with password' credential in Jenkins with ID 'github-token'
                withCredentials([usernamePassword(credentialsId: 'GITHUBTOKEN', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    sh '''
                        # 1. Configure local Git user identity
                        git config user.name "Jenkins CI"
                        git config user.email "jenkins@yourdomain.com"

                        # 2. Move out of detached HEAD state to the main branch
                        git checkout main
                        git pull origin main

                        # 3. Force add the reports directory
                        git add reports/
                        
                        # 4. Only commit if the files have actually changed
                        if ! git diff-index --quiet HEAD --; then
                            git commit -m "chore: update test reports [skip ci]"

                            # 5. Push using the authenticated token URL
                            git push https://${GIT_USER}:${GIT_TOKEN}@://github.com main
                        else
                            echo "No changes found in reports folder. Skipping push."
                        fi
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
