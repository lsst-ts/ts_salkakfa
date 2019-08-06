pipeline {
    agent any
    environment {
        container_name = "c_${BUILD_ID}_${JENKINS_NODE_COOKIE}"
        user_ci = credentials('lsst-io')
    }

    stages {
        stage("Running tests") {
            steps {
                script {
                    sh """
                    docker pull lsstts/salobj:master
                    chmod -R a+rw \${WORKSPACE}
                    container=\$(docker run -v \${WORKSPACE}:/home/saluser/repo/ -td --rm --name \${container_name} -e LTD_USERNAME=\${user_ci_USR} -e LTD_PASSWORD=\${user_ci_PSW} lsstts/salobj:master)
                    docker exec -u saluser \${container_name} sh -c \"source ~/.setup.sh && cd repo && eups declare -r . -t saluser && setup ts_salkafka -t saluser && scons\"
                    """
                }
            }
        }
        stage("Building/Uploading documentation") {
            steps {
                script {
                    sh """
                    docker exec -u saluser \${container_name} sh -c \"source ~/.setup.sh && cd repo && setup ts_salkafka -t saluser && package-docs build && pip install ltd-conveyor==0.5.0a1 && ltd upload --product ts-salkafka --git-ref ${branch_name} --dir doc/_build/html\"
                    """
                }
            }
        }
    }
    post {
        always {
            // The path of xml needed by JUnit is relative to
            // the workspace.
            junit 'tests/.tests/*.xml'

            // Publish the HTML report
            publishHTML (target: [
                allowMissing: false,
                alwaysLinkToLastBuild: false,
                keepAll: true,
                reportDir: 'tests/.tests/pytest-ts_salkafka.xml-htmlcov/',
                reportFiles: 'index.html',
                reportName: "Coverage Report"
              ])
        }
        cleanup {
            sh """
                docker exec -u root --privileged \${container_name} sh -c \"chmod -R a+rw /home/saluser/repo/ \"
                docker stop \${container_name}
            """
            deleteDir()
        }
    }
}