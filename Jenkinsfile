@Library('jenkins-shared-library') _


pipeline {
    agent none
     parameters {
        choice(
            choices: ['DEV','PREPROD', 'PROD'], 
            name: 'Environment'
          )
     }
    stages {
        stage('Check bash syntax') {
            agent { docker { image 'koalaman/shellcheck-alpine:latest' } }
            steps {
              script { bashCheck }
            }
        }
        stage('Check yaml syntax') {
            agent { docker { image 'sdesbure/yamllint' } }
            steps {
              script { yamlCheck }
            }
        }
         
         stage('Prepare ansible environment') {
            agent any
            environment {
                VAULTKEY = credentials('vaultkey')
                DEVOPSKEY = credentials('devopskey')
            }
            steps {
                sh 'echo \$VAULTKEY > vault.key'
                sh 'cp \$DEVOPSKEY id_rsa'
                sh 'chmod 600 id_rsa'
            }
         }
         stage('Test and deploy the application') {
            agent none
            stages {
               stage("Install ansible role dependencies") {
            agent any
                   steps {
                       sh 'ansible-galaxy install  -r roles/requirements.yml'
                   }
               }
                stage("Ping targeted hosts") {
             agent any
                   steps {
                       sh 'ansible all -m ping -i hosts --private-key id_rsa'
                   }
                }

                 stage("Vérify ansible playbook syntax") {
                   agent any
                   steps {
                       sh 'sudo yum install epel-release -y'
                       sh 'sudo yum install python-pip -y'
                       sh 'sudo pip install ansible-lint'
                       sh 'ansible-lint -x 306 phonebook.yml'
                       sh 'echo "${GIT_BRANCH}"'
                   }
                 }
                stage("Installation of configuration files") {
                  agent any
                   steps {
                       sh 'ansible-playbook  -i hosts  security_test.yml'
                   }
                } 
               stage("Build docker images on build host") {
              agent any
                    when{  
                       expression {
                          params.Environment == 'DEV' }
                    }
                   steps {
                       sh 'ansible-playbook  -i hosts --vault-password-file vault.key --private-key id_rsa --tags "build" --limit build phonebook.yml'
                   }
               }


               stage("Scan docker images on build host") {
               agent any
                    when{  
                      expression {
                           params.Environment == 'DEV' }
                    }
                   steps {
                       sh 'ansible-playbook  -i hosts --vault-password-file vault.key --private-key id_rsa --limit build  clair-scan.yml'
                   }

               }
                stage("Push on Artifactory registry") {
              agent any
                    when{  
                      expression {
                         params.Environment == 'DEV' }
                    }
                   steps {
                       sh 'ansible-playbook  -i hosts --vault-password-file vault.key --private-key id_rsa --tags "push" --limit build phonebook.yml'
                   }
                }
 

               stage("Deploy app in Pre-production Environment") {
               agent any
                     when{  
                       expression {
                            params.Environment == 'PREPROD' }
                     }
                   steps {
                       sh 'ansible-playbook  -i hosts --vault-password-file vault.key --private-key id_rsa --tags "preprod" --limit preprod phonebook.yml'
                   }
               }


               stage("Test the functioning of the app in Preprod environment") {
               agent any
                    when{  
                      expression {
                          params.Environment == 'PREPROD' }
                    }
                   steps {
                       sh 'ansible-playbook  -i hosts --vault-password-file vault.key --private-key id_rsa --tags "test" --limit preprod phonebook.yml'
                   }
               }

            }

         }

               stage("Deploy app in Production Environment") {
                    
                 agent any
                    when{  
                      expression {
                          params.Environment == 'PROD' }
                    }

                       sh 'ansible-playbook  -i hosts --vault-password-file vault.key --private-key id_rsa --tags "prod" --limit prod phonebook.yml'
                  
                   }
               }
           /*
           stage('Find xss vulnerability') {
            agent { docker { 
                  image 'gauntlt/gauntlt' 
                  args '-v ${WORKSPACE}:${WORKSPACE}/project --entrypoint='
                  } }
            steps {
                sh 'gauntlt --version'
                sh 'gauntlt ${WORKSPACE}/project/attack/xss.attack'
            }
          }
*/
/*          stage('Find Nmap vulnerability') {
            agent { docker {
                  image 'gauntlt/gauntlt'
                  args '--entrypoint='
                  } }
            steps {
                sh 'gauntlt --version'
                sh 'gauntlt /tmp/nmap.attack'
            }
          }


          stage('Find Os detection vulnerability') {
            agent { docker {
                  image 'gauntlt/gauntlt'
                  args '-v /tmp/attack:${WORKSPACE}/attack --entrypoint='
                  } }
            steps {
                sh 'gauntlt --version'
                sh 'gauntlt /tmp/os_detection.attack'
            }
          }
    }
    post {
     always {
       script {

         clean
        slackNotifier_ops  currentBuild.result
     }
    }

}   
}
