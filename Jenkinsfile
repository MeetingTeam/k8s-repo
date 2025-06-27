def mainBranch = 'main'
def applicationFolder = 'application'
def k8SRepoName = 'k8s-repo'
def githubAccount = 'github'

pipeline {
    agent {
        kubernetes {
            inheritFrom 'default'
        }
    }

    parameters {
        string(
            name: 'SERVICES',
            defaultValue: '',
            description: 'Choose one or multiple services to deploy. Services supported are: chat-service,meeting-service,team-service,user-service,websocket-service'
        )
    }

    stages {
        stage('Clone k8s-repo') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: githubAccount,
                        passwordVariable: 'GIT_PASS',
                        usernameVariable: 'GIT_USER'
                    )
                ]) {
                    sh """
                        git clone https://\${GIT_USER}:\${GIT_PASS}@github.com/MeetingTeam/${k8SRepoName}.git --branch ${mainBranch}
                    """
                }
            }
        }

        stage('Copy image tag') {
            steps {
                script {
                    def selectedServices = params.SERVICES.split(',').collect { it.trim() }.findAll { it }
                    for (service in selectedServices) {
                        def devValuesFile = "${k8SRepoName}/${applicationFolder}/${service}/values.dev.yaml"
                        def prodValuesFile = "${k8SRepoName}/${applicationFolder}/${service}/values.prod.yaml"
                        sh """
                            image_tag=\$(sed -n 's/.*imageTag: *\\(.*\\)/\\1/p' ${devValuesFile})
                            sed -i "/imageTag:/s/:.*/: \$image_tag/" ${prodValuesFile}
                        """
                        if (service == "user-service" || service == "team-service") {
                            sh """
                                job_image_tag=\$(sed -n 's/.*jobImageTag: *\\(.*\\)/\\1/p' ${devValuesFile})
                                sed -i "/imageTag:/s/:.*/: \$job_image_tag/" ${prodValuesFile}
                            """
                        }
                    }
                }
            }
        }

        stage('Commit and push changes') {
            steps {
                sh """
                    cd ${k8SRepoName}
                    git config --global user.email "jenkins@gmail.com"
                    git config --global user.name "Jenkins"
                    git add .
                    git commit -m "chore: sync prod image tags from test environment for ${params.SERVICES} services"
                    git push origin ${mainBranch}
                """
            }
        }
    }
}