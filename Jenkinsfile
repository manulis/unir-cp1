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
                
        stage('Coverage') {
            steps {

                sh '''
                    python3 -m coverage run --branch --source=app --omit=app/__init__.py,app/api.py -m pytest test/unit
                    python3 -m coverage xml -o coverage.xml
                '''

                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    cobertura coberturaReportFile: 'coverage.xml',  
                             conditionalCoverageTargets: '100,0,83', 
                             lineCoverageTargets: '100,0,90'
                }
            }
        }
        
        stage('Static') {
            steps {

                sh '''
                    python3 -m flake8 --exit-zero --format=pylint app > flake8.out
                '''

                recordIssues(
                    tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], 
                    qualityGates: [
                        [threshold: 10, type: 'TOTAL', unstable: true], 
                        [threshold: 11, type: 'TOTAL', unstable: true]
                    ]
                )
            }
        }
        
        stage('Security') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh '''
                        python3 -m bandit -r . -f custom -o bandit.out --msg-template "{abspath}:{line}: {severity}: {test_id}: {msg}"
                    '''
                }
                recordIssues(
                    qualityGates: [
                        [threshold: 4, type: 'TOTAL', unstable: true],
                        [threshold: 8, type: 'TOTAL', unstable: true]
                    ],
                    tools: [pyLint(pattern: 'bandit.out', name: 'Bandit')]
                )
                
            }
        }


        
        stage('Performance') {
            steps {
                sh '''
                    export FLASK_APP=app/api.py
                   /opt/apache-jmeter-5.5/bin/jmeter -n -t test/jmeter/flask.jmx -l flask.jtl
                '''
                perfReport sourceDataFiles: 'flask.jtl' 
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