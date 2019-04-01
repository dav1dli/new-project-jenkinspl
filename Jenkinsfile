// This script requires evaluated cluster permissions allowing new projects creation,
// and security content, quota, and accounts manipulation
// oc adm policy add-cluster-role-to-user cluster-admin system:serviceaccount:newprj-demo:jenkins --as=system:admin
pipeline {
  agent any
  options {
    timeout(time: 20, unit: 'MINUTES')
  }
  parameters {
    string(name: 'projectname', description: 'OpenShift Project Name', defaultValue: "tif-project")
    string(name: 'projectdescription', defaultValue: "", description: 'OpenShift Project Description')
    string(name: 'projectdisplayname', defaultValue: "", description: 'OpenShift Project DisplayName')
    string(name: 'projectadmin', description: 'Project Admin', defaultValue: "admin")
    choice(name: 'projectsize', choices: ['s', 'm', 'l', 'z'], description: 'Project Size, z is zero (just for projects with config files in it)')
    booleanParam(name: 'api_management', defaultValue: true, description: 'Is the project exposing or consuming APIs from TAPIR?')
    string(name: 'customlabel', defaultValue: 'tif.telekom.de/businessproject=tif-development', description: 'Add a custom label, e.g. tif.telekom.de/businessproject=tif-development')
    booleanParam(name: 'dry_run', defaultValue: true, description: 'Show commands but not execute')
  }
  stages {
    stage ('CheckParameters') {
      when {
        expression {
          "${params.projectname}" == "" || "${params.projectadmin}" == ""
        }
      }
      steps {
        error("projectname and projectadmin must be provided")
      }
    }
    stage ('NewProject') {
      steps {
        script {
          displayName = "${params.projectdisplayname}" == "" ? "${params.projectname}" : "${params.projectdisplayname}"
          description = "${params.projectdescription}" == "" ? "${params.projectname}" : "${params.projectdescription}"
          env.PROJECT = "${params.projectname}"
          if (params.dry_run) {
            echo "Dry run: commands displayed but not executed"
            sh 'pwd'
            sh 'find .'
            sh 'ls -lFa'
          }
        }
        script {
          openshift.withCluster() {
            def int exitCode = sh(script: "oc get project ${PROJECT} 2>/dev/null",
              returnStatus: true)
            if(exitCode != 0) {
              echo "Project ${params.projectname} was not found"
              if (params.dry_run) {
                echo "Dry run: oc adm new-project ${params.projectname} \
                  --description='${description}' \
                  --display-name='${displayName}' \
                  --admin='${params.projectadmin}'"
               } else {
                sh "oc adm new-project ${params.projectname} \
                  --description='${description}' \
                  --display-name='${displayName}' \
                  --admin='${params.projectadmin}'"
              }
            } else {
              echo "Project ${params.projectname} already exists"
            }
            if (params.dry_run) {
              echo "Dry run: oc label namespace ${params.projectname} projectsize=${params.projectsize} ${params.customlabel}"
             } else {
              sh "oc label namespace ${params.projectname} projectsize=${params.projectsize} ${params.customlabel}"
            }
          }
        }
      }
    }
    stage('CreateQuota') {
      steps {
        script {
          openshift.withCluster() {
            if (params.dry_run) {
              echo "Dry run: git clone https://github.com/dav1dli/new-project-jenkinspl.git ."
              echo "Dry run: oc create -f templates/resource_template_size_${params.projectsize}.yml -n ${params.projectname}"
             } else {
               git url: 'https://github.com/dav1dli/new-project-jenkinspl.git'
               sh "oc create -f templates/resource_template_size_${params.projectsize}.yml -n ${params.projectname}"
            }
          }
        }
      }
    }
    stage('Add SCC') {
      steps {
        script {
          openshift.withCluster() {
            if (params.dry_run) {
              echo "Dry run: oc adm policy add-scc-to-user privileged system:serviceaccount:${params.projectname}:default"
             } else {
               sh "oc adm policy add-scc-to-user privileged system:serviceaccount:${params.projectname}:default"
            }
          }
        }
      }
    }
    stage('Create Pipeline SA') {
      steps {
        script {
          openshift.withCluster() {
            if (params.dry_run) {
              echo "Dry run: oc create serviceaccount tif-deployer -n ${params.projectname}"
             } else {
               sh "oc create serviceaccount tif-deployer -n ${params.projectname}"
            }
          }
        }
      }
    }
    stage('Add edit rights to SA') {
      steps {
        script {
          openshift.withCluster() {
            if (params.dry_run) {
              echo "Dry run: oc adm policy add-role-to-user edit system:serviceaccount:${params.projectname}:tif-deployer -n ${params.projectname}"
             } else {
               sh "oc adm policy add-role-to-user edit system:serviceaccount:${params.projectname}:tif-deployer -n ${params.projectname}"
            }
          }
        }
      }
    }
  }
}
