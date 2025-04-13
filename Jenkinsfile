def mainBranch = 'main'
def applicationFolder = 'application'

pipeline {
    agent {
        kubernetes {
            inheritFrom 'yp'
        }
    }

    parameters {
        booleanParam(name: 'CHAT_SERVICE', defaultValue: false, description: 'Deploy chat service')
        booleanParam(name: 'MEETING_SERVICE', defaultValue: false, description: 'Deploy meeting service')
        booleanParam(name: 'USER_SERVICE', defaultValue: false, description: 'Deploy user service')
        booleanParam(name: 'TEAM_SERVICE', defaultValue: false, description: 'Deploy team service')
        booleanParam(name: 'WEBSOCKET_SERVICE', defaultValue: false, description: 'Deploy websocket service')
    }

    stages {
          stage('Copy image tag') {
            steps {
                container('yp') {
                    script {
                        def selectedServices = []
                        if (params.CHAT_SERVICE) selectedServices << "chat-service"
                        if (params.MEETING_SERVICE) selectedServices << "meeting-service"
                        if (params.USER_SERVICE) selectedServices << "user-service"
                        if (params.TEAM_SERVICE) selectedServices << "team-service"
                        if (params.WEBSOCKET_SERVICE) selectedServices << "websocket-service"

                        // Iterate over selected services and copy the image tag
                        for (chart in selectedServices) {
                            sh """
                                chart_dir=${applicationFolder}/${chart}
                                test_file=\$chart_dir/values.test.yaml
                                prod_file=\$chart_dir/values.prod.yaml

                                image_tag=\$(yq '.image.tag' \$test_file)
                                yq -i ".image.tag = \\"\$image_tag\\"" \$prod_file

                                job_image_tag=\$(yq '.job.image.tag' \$test_file)
                                yq -i ".job.image.tag = \\"\$job_image_tag\\"" \$prod_file
                            """
                        }
                    }
                }
            }
          }
          stage('Commit and push changes') {
            steps {
                sh """
                    git config --global user.email "jenkins@gmail.com"
                    git config --global user.name "Jenkins"
                    git add .
                    git commit -m "chore: sync prod image tag(s) from test for ${selectedServices.join(', ')}"
                    git push origin ${mainBranch}
                """
            }
        }
    }
}