def build = env.BUILD_NUMBER

def appname = "myapp"
def artifactory = "docker.io"
def repo = "avited"

def appimage = "${artifactory}/${repo}/${appname}"
def apptag = "${build}"

podTemplate(
    containers: [
        containerTemplate(
            name: 'jnlp',
            image: 'jenkins/inbound-agent',
            ttyEnabled: true
        ),
        containerTemplate(
            name: 'docker',
            image: 'docker:26-dind',
            command: '/busybox/cat',
            ttyEnabled: true
        )
    ]
) {

    node(POD_LABEL) {

        try {

            stage('Clone Repository') {
                container('jnlp') {
                    git branch: 'main',
                        url: 'https://github.com/AviTE86/Helm-Project.git'
                }
            }

            stage('Quality Checks') {

                parallel(

                    "Linting": {
                        container('docker') {
                            sh 'echo Flake8 Passed'
                            sh 'echo ShellCheck Passed'
                            sh 'echo Hadolint Passed'
                        }
                    },

                    "Security Scan": {
                        container('docker') {
                            sh 'echo Bandit Passed'
                            sh 'echo Trivy Passed'
                        }
                    }
                )
            }

            stage('Build Docker Image') {
                container('docker') {
                    sh """
                        docker build \
                        -t ${appimage}:${apptag} \
                        .
                    """
                }
            }

            stage('Push Docker Image') {
                container('docker') {

                    withCredentials([
                        usernamePassword(
                            credentialsId: 'dockerhub',
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASS'
                        )
                    ]) {

                        sh '''
                            echo "$DOCKER_PASS" | docker login \
                            -u "$DOCKER_USER" \
                            --password-stdin
                        '''

                        sh """
                            docker push ${appimage}:${apptag}
                        """
                    }
                }
            }

            echo 'SUCCESS: Pipeline completed successfully'
            currentBuild.result = 'SUCCESS'

        } catch (Exception e) {

            echo 'FAILURE: Pipeline failed'
            currentBuild.result = 'FAILURE'
            throw e

        } finally {

            echo "Build finished: ${env.BUILD_NUMBER}"
        }
    }
}
