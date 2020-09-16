pipeline{
  agent any
  environment {
    // 環境にあわせて変更してください
    PIP_NAMESPACE='eap-demo-devpipline'
    STG_NAMESPACE='eap-demo-stg'
    PRD_NAMESPACE='eap-demo-prd'
    APP_NAME='eap-app'
  }
  parameters {
    string(name: 'tag', defaultValue: 'latest', description: 'Image Tag Name')
  }
  stages{
    stage("Deploy ImageStream to Staging"){
      when {
        expression {
          openshift.withCluster() {
            openshift.withProject('${STG_NAMESPACE}'){
              return !openshift.selector('imagestream', '${APP_NAME}').exists();
            }
          }
        }
      }
      steps{
        script{
          openshift.withCluster(){
            openshift.withProject('${STG_NAMESPACE}'){
              openshift.create('imagestream', '${APP_NAME}')
            }
          }
        }
      }
    }
    stage("Promote Docker Image to Staging"){
      steps{
        script{
          openshift.withCluster(){
            openshift.withProject('${STG_NAMESPACE}'){
              openshift.tag('${PIP_NAMESPACE}/${APP_NAME}:$tag',
                            '${STG_NAMESPACE}/${APP_NAME}:$tag-stg')
            }
          }
        }
      }
    }
    stage("Deploy Staging"){
      steps{
        script{
          openshift.withCluster(){
            openshift.withProject('${STG_NAMESPACE}'){
              def manifests = findFiles(glob: 'stg/*.yaml')
              manifests.each{ item ->
                openshift.apply(readFile("stg/${item.name}"))
              }
            }
          }
        }  
      }
    }
    stage("Verify Staging Deployment"){
      steps{
        script{
          openshift.withCluster() {
            openshift.withProject( "${STG_NAMESPACE}" ){
              timeout(5){
              // Check Replicas
                waitUntil{
                  ! openshift.selector('deployment',"${APP_NAME}").object().status.unavailableReplicas
                }
                // Connect to Service
                waitUntil{
                  openshift.verifyService("${APP_NAME}")
                }
              }
            }
          }
        }
      }
    }
    stage("Test"){
      steps{
        echo "Wirte Test here"
      }
    }
    stage("Approve"){
      steps{
        script {
          input(
             id: "prd-approver",
             message: "本番環境にデプロイを開始します。", 
             parameters: [
               string(defaultValue: '', description: '承認者の名前を入れてください', name: '承認者')
             ]
          )
        }
      }
    }
    stage("Deploy ImageStream to Production"){
      when {
        expression {
          openshift.withCluster() {
            openshift.withProject('${PRD_NAMESPACE}'){
              return !openshift.selector('imagestream', '${APP_NAME}').exists();
            }
          }
        }
      }
      steps{
        script{
          openshift.withCluster(){
            openshift.withProject('${PRD_NAMESPACE}'){
              openshift.create('imagestream', '${APP_NAME}')
            }
          }
        }
      }
    }
    stage("Promote Docker Image to Production"){
      steps{
        script{
          openshift.withCluster(){
            openshift.withProject('${PRD_NAMESPACE}'){
              openshift.tag('${STG_NAMESPACE}/${APP_NAME}:$tag-stg',
                            '${PRD_NAMESPACE}/${APP_NAME}:$tag-prd')
            }
          }
        }
      }
    }
    stage("Deploy Production"){
      steps{
        script{
          openshift.withCluster(){
            openshift.withProject('${PRD_NAMESPACE}'){
              def manifests = findFiles(glob: 'prd/*.yaml')
              manifests.each{ item ->
                openshift.apply(readFile("prd/${item.name}"))
              }
            }
          }
        }
      }
    }
    stage("Verify Production Deployment"){
      steps{
        script{
          openshift.withCluster() {
            openshift.withProject( "${PRD_NAMESPACE}" ){
              timeout(5){
              // Check Replicas
                waitUntil{
                  ! openshift.selector('deployment',"${APP_NAME}").object().status.unavailableReplicas
                }
                // Connect to Service
                waitUntil{
                  openshift.verifyService("${APP_NAME}")
                }
              }
            }
          }
        }
      }
    }
  }
}
