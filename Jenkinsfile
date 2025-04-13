def mainBranch = 'main'
def applicationFolder = 'application'


properties([
          parameters([[
                    $class: 'ExtendedChoiceParameterDefinition',
                    name: 'SELECTED_SERVICES',
                    type: 'PT_CHECKBOX',
                    multiSelectDelimiter: ',',
                    quoteValue: false,
                    description: 'Select one or more services to deploy',
                    choices: 'chat-service\nmeeting-service\nuser-service\nteam-service\nwebsocket-service'
          ]])
])
pipeline{
         agent {
                    kubernetes {
                              inheritFrom 'yp'
                    }
          }
          
          stages{
                    stage('Copy image tag') {
                              steps{
                                        container('yp'){
                                                  script {
                                                            def selectedServices = params.SELECTED_SERVICES.split(',')
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
                                                  git commit -m "chore: sync prod image tag(s) from test for ${params.SELECTED_SERVICES}"
                                                  git push origin ${mainBranch}                                                    
                                        """				
                              }
                    }
          }
}