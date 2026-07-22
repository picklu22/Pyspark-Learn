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
                // Dynamically binds your 'Secret text' credential to the $GIT_TOKEN variable
                withCredentials([string(credentialsId: 'GITHUBTOKEN', variable: 'GIT_TOKEN')]) {
                    sh '''
                        # 1. Configure local Git user identity
                        git config user.name "Jenkins CI"
                        git config user.email "jenkins@yourdomain.com"

                        # 2. Force checkout and sync with remote branch
                        git checkout -f main
                        git config pull.rebase false
                        git pull origin main --strategy-option=theirs --allow-unrelated-histories

                        # 3. Add the reports directory to the staging index
                        git add reports/
                        
                        # 4. Check specifically if anything inside the reports folder is modified or new
                        if [ -n "$(git status --porcelain reports/)" ]; then
                            git commit -m "chore: update test reports [skip ci]"

                            # 5. Push using the authenticated token URL string
                            git push https://picklu22:${GIT_TOKEN}@://github.com main
                        else
                            echo "Reports folder matches upstream exactly. Skipping push."
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
