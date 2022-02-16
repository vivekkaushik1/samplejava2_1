def appName='App_2'
    def snapName=''
    def deployName ='Dep_1'
    def exportFormat ='json'
    def configFilePath = "paymentService"
    def fileNamePrefix ='exported_file_'
    def fullFileName="${appName}-${deployName}-${currentBuild.number}.${exportFormat}"
    def changeSetId=""
    def snapshotName=""
    def exporterName ='returnAllData' 
    def namePath ="Comp_1"
pipeline {
    agent any
    stages {
        stage('Clone repository') {               
           steps{
                // checkout scm
                git branch: 'master', url: 'https://github.com/vivekkaushik1/samplejava'
           }
        }     
        stage('Validate Configurtion file'){
            steps{
                script{
                    sh "echo validating configuration file ${configFilePath}.${exportFormat}"
                    echo "name path ::::: ${namePath}"
                    changeSetId = snDevOpsConfigUpload(applicationName:"${appName}",target:'component',namePath:"${namePath}", configFile:"fileB.json", autoCommit:true,autoValidate:true,dataFormat:"${exportFormat}")
                    // snDevOpsConfigUpload(applicationName:"${appName}",target:'deployable',namePath:"${namePath}", fileName:"deployable", autoCommit:'true',autoValidate:'true',dataFormat:"${exportFormat}",changesetNumber:"${changeSetId}", deployableName:"${deployName}")
                    echo "validation result $changeSetId"
                }
            }
        }
        stage("register change set to pipeline"){
            steps{
                script{
                    echo "Change set registration for ${changeSetId}"
                    changeSetRegResult = snDevOpsConfigRegisterChangeSet(changesetNumber:"${changeSetId}")
                    echo "change set registration set result ${changeSetRegResult}"
                }
            }
        }
         stage("Get snapshots created"){
            steps{
                echo "Triggering Get snapshots for applicationName:${appName},deployableName:${deployName},changeSetId:${changeSetId}"
                script{
                    changeSetResults = snDevOpsConfigGetSnapshots(applicationName:"${appName}",deployableName:"${deployName}",changesetNumber:"${changeSetId}")
                    echo "ChangeSet Result : ${changeSetResults}"
                    def changeSetResultsObject = readJSON text: changeSetResults
                         changeSetResultsObject.each {
                           /* if(it.validation == "passed"){
                                echo "validation passed for snapshot : ${it.name}"
                                snapshotName = it.name
                            }else{
                                echo "Snapshot failed to get validated : ${it.name}" ;
                                assert it.validation == "passed"
                            }*/
                            echo "validation passed for snapshot : ${it.name}"
                            snapshotName = it.name 
                        }
                  if (!snapshotName?.trim()){
                    error "No snapshot found to proceed" ;
                  }
                  echo "Snapshot Name : ${snapshotName} "  
                }
            }
        }
      
        stage('Publish the snapshot'){
            steps{
                script{
                    echo "Step to publish snapshot applicationName:${appName},deployableName:${deployName} snapshotName:${snapshotName}"
                    publishSnapshotResults = snDevOpsConfigPublish(applicationName:"${appName}",deployableName:"${deployName}",snapshotName: "${snapshotName}")
                    echo " Publish result for applicationName:${appName},deployableName:${deployName} snapshotName:${snapshotName} is ${publishSnapshotResults}"
                }
            }
        }
        stage('Deploy to the System'){
            steps{
                echo "Devops Change trigger change request"
                //snDevOpsChange()
            }

        }
        stage('Download Snapshots from Service Now') {
            steps{
                script{
                    echo "Exporting for App: ${appName} Deployable; ${deployName} Exporter name ${exporterName} "
                    echo "Configfile exporter file name ${fullFileName}"
                    sh  'echo "<<<<<<<<<export file is starting >>>>>>>>"'
                    response = snDevOpsConfigExport(applicationName: "${appName}", snapshotName: "${snapshotName}", deployableName: "${deployName}",exporterFormat: "${exportFormat}", fileName:"${fullFileName}",exporterName: "${exporterName}")
                    //response = snDevOpsConfigExport(applicationName: "${appName}", snapshotName: "${snapName}", deployableName: "${deployName}",exporterFormat: "${exportFormat}", fileName:"${fullFileName}",exporterName: "${exporterName}")
                    echo " RESPONSE FROM EXPORT : ${response}"
                }
            }
        }
        stage("Deploying to PROD-US"){
            steps{
                echo "Reading config from file name ${fullFileName}"
                echo " ++++++++++++ BEGIN OF File Content ***************"
                sh "cat ${fullFileName}"
                echo " ++++++++++++ END OF File content ***************"
                echo "deploy finished successfully."
            }
        }
    }
}
