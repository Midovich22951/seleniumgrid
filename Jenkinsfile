def templatePath = 'https://raw.githubusercontent.com/Midovich22951/seleniumgrid/master/selenium-hub.yaml'
def templatePath1 = 'https://raw.githubusercontent.com/Midovich22951/seleniumgrid/master/selenium-node-template.yaml'

def templateName = 'selenium-hub' 
def templateName1 = 'selenium-node-chrome'
pipeline {
  agent {
    node {
      label 'master' 
    }
  }
  options {
    timeout(time: 20, unit: 'MINUTES') 
  }
  stages {
    stage('preamble') {
        steps {
            script {
                openshift.withCluster() {
                    openshift.withProject() {
                        echo "Using project: ${openshift.project()}"
                    }
                }
            }
        }
    }
    stage('cleanup selenium grid') {
      steps {
        script {
            openshift.withCluster() {
                        openshift.withProject() {
                            // delete everything with this template label
                            //openshift.selector('all', [ app : templateName ]).delete()
                            openshift.selector('route', [ app : templateName ]).delete()
                            openshift.selector('svc', [ app : templateName ]).delete()
                            openshift.selector('dc', [ app : templateName ]).delete()
                          
                            openshift.selector('route', [ app : templateName1 ]).delete()
                            openshift.selector('svc', [ app : templateName1 ]).delete()
                            openshift.selector('dc', [ app : templateName1 ]).delete()
                    }
                  }
                }
              }
           }
    
    stage('create Hub') {
      steps {
        script {
            openshift.withCluster() {
                openshift.withProject() {
                  openshift.newApp(templatePath) 
                }
            }
        }
      }
    }
    
    stage('create Nodes') {
      steps {
        script {
            openshift.withCluster() {
                openshift.withProject() {
                  openshift.newApp(templatePath1) 
                }
            }
        }
      }
    }
    
    
    stage('build Hub') {
      steps {
        script {
            openshift.withCluster() {
                openshift.withProject() {
                  def builds = openshift.selector("bc", templateName).related('builds')
                  timeout(5) { 
                    builds.untilEach(1) {
                      return (it.object().status.phase == "Complete")
                    }
                  }
                }
               
            }
        }
      }
    }
    stage('build Nodes') {
      steps {
        script {
            openshift.withCluster() {
                openshift.withProject() {
                  def builds1 = openshift.selector("bc", templateName1).related('builds')
                  timeout(5) { 
                    builds1.untilEach(1) {
                      return (it.object().status.phase == "Complete")
                    }
                  }
                }
               
            }
        }
      }
    }
    stage('deploy Hub') {
      steps {
        script {
            openshift.withCluster() {
                openshift.withProject() {
                  def rm = openshift.selector("dc", templateName).rollout()
                  timeout(5) { 
                    openshift.selector("dc", templateName).related('pods').untilEach(1) {
                      return (it.object().status.phase == "Running")
                    }
                  }
                }
            }
        }
      }
    }
    
    stage('deploy nodes') {
      steps {
        script {
            openshift.withCluster() {
                openshift.withProject() {
                  def rm1 = openshift.selector("dc", templateName1).rollout()
                  timeout(5) { 
                    openshift.selector("dc", templateName1).related('pods').untilEach(1) {
                      return (it.object().status.phase == "Running")
                    }
                  }
                }
            }
        }
      }
    }
    stage('tag Hub') {
      steps {
        script {
            openshift.withCluster() {
                openshift.withProject() {
                  openshift.tag("${templateName}:latest", "${templateName}-staging:latest") 
                }
            }
        }
      }
    }
    
     stage('tag Nodes') {
      steps {
        script {
            openshift.withCluster() {
                openshift.withProject() {
                  openshift.tag("${templateName}:latest", "${templateName}-staging:latest") 
                }
            }
        }
      }
    }
    
  }
}
