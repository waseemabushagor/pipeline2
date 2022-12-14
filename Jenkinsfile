pipeline {
    agent any
    environment {
       DEP_COLOR = 'BLUE'
    }
    options {
        skipDefaultCheckout()
    }
    stages {
        stage('Checking for Deployment type.') {
            steps {
                script {
                    RESULTS = sh (script: "git log -1 | grep '\\[GREEN\\]'", returnStatus: true)
                    if (RESULTS == 0) {
                        DEP_COLOR = "GREEN"
                    }
                }
            }
        }
        stage('Building') {
            steps {
                echo "Building.."
                sh '''
                cd flask-calculator
                pip3 install -r requirements.txt
                '''
            }
        }
        stage('Test') {
            steps {
                echo "Testing.."
                sh '''
                cd flask-calculator
                python3 -m unittest TestCalc.py
                '''
            }
        }
        stage('Docker Build') {
            steps {
                echo 'Building docker image from Dockerfile....'
                sh '''
                cd flask-calculator
                docker build -t waseemabushagor/flask-app:$BUILD_NUMBER .
                '''
            }
        }
        stage('Deliver') {
            steps {
                withDockerRegistry([ credentialsId: "dockerhub", url: "" ]) {
                    sh  'docker push waseemabushagord/flask-app:$BUILD_NUMBER'
                    echo 'Run docker container'
                    // sh 'docker kill $(docker ps -q)'
                    // sh 'docker run -d -p 3000:3000 mshmsudd/flask-app'
                }
            }
        }
        stage('blue kubernetes deployment')  {
            steps {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-cred', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                        cd flask-calculator-deployment
                        kubectl apply -f k8s-flask-calculator-deployment.yml
                    '''
                }
            }
        }
        stage('green kubernetes deployment') {
            steps {

                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-cred', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                        cd flask-calculator-deployment
                        kubectl apply -f k8s-flask-calculator-deployment.yml
                    '''
                }   
            }
        }
    }
    '''post {
        success {
            discordSend description: "Jenkins Pipeline Build", footer: "Footer Text", link: env.BUILD_URL, result: currentBuild.currentResult, title: JOB_NAME, webhookURL: ""
        }
    }'''
}
