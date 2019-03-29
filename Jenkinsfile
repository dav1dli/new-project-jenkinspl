pipeline {
  agent any
  options {
    timeout(time: 20, unit: 'MINUTES')
  }
  parameters {
    string(name: 'projectname', description: 'OpenShift Project Name')
    string(name: 'projectdescription', defaultValue: "", description: 'OpenShift Project Description')
    string(name: 'projectdisplayname', defaultValue: "", description: 'OpenShift Project DisplayName')
    string(name: 'projectadmin', description: 'Project Admin')
    choice(name: 'projectsize', choices: ['s', 'm', 'l', 'z'], description: 'Project Size, z is zero (just for projects with config files in it)')
    booleanParam(name: 'api_management', defaultValue: true, description: 'Is the project exposing or consuming APIs from TAPIR?')
    string(name: 'customlabel', defaultValue: 'tif.telekom.de/businessproject=tif-development', description: 'Add a custom label, e.g. tif.telekom.de/businessproject=tif-development')
  }
  stages {
    stage('Check params') {
      when {
        expression {
          "${params.projectname}" == "" || "${params.projectadmin}" == ""
        }
      }
      steps {
        error("projectname and projectadmin must be provided")
      }
    }
    stage('Create project') {
      steps {
        script {
          displayName = "${params.projectdisplayname}" == "" ? "${params.projectname}" : "${params.projectdisplayname}"
          description = "${params.projectdescription}" == "" ? "${params.projectname}" : "${params.projectdescription}"
        }
        sh "oc adm new-project ${params.projectname} --description='${description}' --display-name='${displayName}' --admin='${params.projectadmin}'"
        sh "oc label namespace ${params.projectname} projectsize=${params.projectsize} ${params.customlabel}"
      }
    }
    stage('Create Quota') {
      steps {
        echo "to be done, checkout from repo and put files there";
      }
    }
    stage('Add SCC') {
      steps {
        sh "oc adm policy add-scc-to-user privileged system:serviceaccount:${params.projectname}:default"
      }
    }
    stage('Create Pipeline SA') {
      steps {
        sh "oc create serviceaccount tif-deployer -n ${params.projectname}"
      }
    }
    stage('Add edit rights to SA') {
      steps {
        sh "oc adm policy add-role-to-user edit system:serviceaccount:${params.projectname}:tif-deployer -n ${params.projectname}"
      }
    }
  }
}
