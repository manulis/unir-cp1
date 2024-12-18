pipeline {
    agent any

    stages {
        stage('Perform Preparation') {
            steps {
                script {
                    echo "Get Code"
                    sh '''
                        git clone https://github.com/manulis/unir-cp1.git
                        ls
                    '''
                }
            }
        }
        stage('Build') {
            steps {
                echo 'This is python no build needed'
            }
        }
        stage('Tests') {
            parallel {
                stage('Unit') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            sh '''
                                if [ ! -d "test/unit" ]; then
                                    echo "Directory test/unit does not exist"
                                    exit 1
                                fi
                                export PYTHONPATH=.
                                pytest --junitxml=result-unit.xml test/unit
                            '''
                        }
                    }
                }
                stage('Rest') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            sh '''
                                if [ ! -d "test/rest" ]; then
                                    echo "Directory test/rest does not exist"
                                    exit 1
                                fi
                                
                                export FLASK_APP=app/api.py
                                flask run &
                                java -jar /usr/local/bin/wiremock.jar --port 9090 --verbose --root-dir test/wiremock &
                                

                                until curl --silent --max-time 10 http://localhost:5000 > /dev/null; do
                                    echo "Waiting for Flask (port 5000) to be ready..."
                                    sleep 2
                                done

    
                                until curl --silent --max-time 10 http://localhost:9090 > /dev/null; do
                                    echo "Waiting for WireMock (port 9090) to be ready..."
                                    sleep 2
                                done

                                
                                echo "Flash and Wiremock are ready, starting the tests"
                                pytest --junitxml=result-rest.xml test/rest
                            '''
                        }
                    }
                }
            }
        }
        stage('Result') {
            steps {
                junit "result*.xml"
            }
        }
    }

    post {
        always {
            echo "Cleaning workspace"
            deleteDir()
        }
    }
}
