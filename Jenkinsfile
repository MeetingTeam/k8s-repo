def mainBranch = 'main'
def applicationFolder = 'application'

pipeline {
          agent {
                    kubernetes {
                              inheritFrom 'yp'
                    }
          }

          parameters {
                    string(name: 'SERVICES', defaultValue: '', description: 'Choose one or multiple services to deploy. Services supported are: chat-service,meeting-service,team-service,user-service,websocket-service')
          }

          stages {
                    stage('Copy image tag') {
                              steps {
                                        container('yp') {
                                                  script {
                                                            def selectedServices = params.SERVICES.split(',')
                                                            for(chart in selectedServices) {
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
                                                  git commit -m "chore: sync prod image tags from test environment for ${params.SERVICES} services"
                                                  git push origin ${mainBranch}
                                        """
                              }
                    }
          }
}