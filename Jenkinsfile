pipeline {
    agent {
        label 'soumik-EC2'
    }
    environment {
        MONGO_URI = credentials('soumik-mongo-uri')
        SECRET_KEY = credentials('soumik-mongo-secretkey')
    }
    stages {
        stage('checkout') {
            steps {
                git url: 'https://github.com/soumik5/flask_Practice.git',
                branch: 'main'   
                }
            }

        stage('install dependencies') {
            steps {
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip3 install -r requirements.txt
                printf "MONGO_URI=$MONGO_URI\nSECRET_KEY=$SECRET_KEY" > .env
                '''
            }       
        }
        stage('test') {
            steps {
                sh '''
                . venv/bin/activate
                pytest test_app.py
                '''
            }
        }

        stage('create systemd service for flaskapp') {
            steps {
                sh '''
                cat > flaskapp.service <<EOF
                [Unit]
                Description=Flask Application Service
                
                [Service]
                User=ubuntu
                WorkingDirectory=/python-app
                EnvironmentFile=/python-app/.env
                ExecStart=/python-app/venv/bin/python3 /python-app/app.py
                Restart=always
                RestartSec=5

                [Install]
                WantedBy=multi-user.target
                EOF
                '''
                
            }
        }

        stage('deploy') {
            steps {
                sh '''
                sudo mv flaskapp.service /etc/systemd/system/flaskapp.service
                sudo ls -l /etc/systemd/system/flaskapp.service
                sudo mkdir -p /python-app
                sudo rsync -a --delete $WORKSPACE/ /python-app/
                sudo systemctl daemon-reload
                sudo systemctl enable flaskapp
                sudo systemctl restart flaskapp
                sudo systemctl status flaskapp --no-pager
                sleep 5
                ps -ef | grep app.py | grep -v grep
                '''
            }
        }
    
    }

    post {

        success {
            emailext(
                to: 's4soumik.chanda@gmail.com',
                subject: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
    Hello,
    
    The Jenkins pipeline completed successfully.
    
    Job Name     : ${env.JOB_NAME}
    Build Number : ${env.BUILD_NUMBER}
    Status       : SUCCESS
    
    The Flask application was:
    ✔ Checked out
    ✔ Built
    ✔ Tested
    ✔ Deployed successfully
    ✔ Verified Deployment
    
    Build URL:
    ${env.BUILD_URL}
    
    Regards,
    Jenkins CI/CD
    """
            )
        }
    
        failure {
            emailext(
                to: 's4soumik.chanda@gmail.com',
                subject: "❌ FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
    Hello,
    
    The Jenkins pipeline has failed.
    
    Job Name     : ${env.JOB_NAME}
    Build Number : ${env.BUILD_NUMBER}
    Status       : FAILURE
    
    Please review the console output for error details.
    
    Build URL:
    ${env.BUILD_URL}
    
    Regards,
    Jenkins CI/CD
    """
            )
        }
    }
}
