pipeline {
    agent any

    stages {
        stage('Perform Preparation') {
            steps {
                script {
                    echo "Cleaning workspace"
                    deleteDir()
                    echo "Get Code"
                    sh '''
                        git clone https://github.com/manulis/helloworld.git
                        ls
                    '''
                }
            }
        }
        stage('Build'){
            steps{
                echo ""
            }
        }
        stage('Tests'){
            parallel{
                stage('Unit') {
                        steps {
                            catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                                sh '''
                                    set -e
                                    cd helloworld
                                    if [ ! -d "test/unit" ]; then
                                        echo "Directory test/unit does not exist"
                                        exit 1
                                    fi
                                    export PYTHONPATH=.
                                    pytest --junitxml=../result-unit.xml test/unit
                                '''
                            }
                        }
                    }
                stage('Rest'){
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            sh '''
                                set -e
                                cd helloworld
                                if [ ! -d "test/unit" ]; then
                                    echo "Directory test/unit does not exist"
                                    exit 1
                                fi
                                export FLASK_APP=app/api.py 
                                flask run &
                                pytest --junitxml=../result-rest.xml test/rest
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
}