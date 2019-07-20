pipeline {
  agent any
  
	stages {
	    stage('Pipeline Start') {		
			steps {
			   echo 'Pipeline Started with Vars Declared'
			}
	    }
	}
}
node {
	def nexus_Protocol = ""
	def nexus_BaseURL = ""		
	def nexus_RepoName = ""		
	def pom_GroupID = ""
	pom_ArtifactId = ""
	def pom_Version = ""
	def pom_Packaging = ""
	def downloadDir = ""

	properties([
     parameters([
       choiceParam(
         choices: 'DEV\nSIT\nUAT\nPROD',
            description: 'Please select the Build ENVIRONMENTS',
            name: 'ENVIRONMENTS'
       ),
       choiceParam(
         choices: 'BUILD\nBUILD-AND-ARTEFACT-UPLOAD\nBUILD-AND-RELEASE\nRELEASE',
            description: 'Please select the Build Mechanism',
            name: 'BUILD_MECHANISM'
       ),
       choiceParam(
         choices: 'YES\nNO',
            description: 'Is application existed on cloud?/Are you re-deploying application on Cloud?',
            name: 'APP_EXISTS'
       ),
	   string(
	   name: 'BuildParameters', 
	   defaultValue: 'mvn -U install -DskipTests=true', 
	   description: 'Please provide Additional Build Parameter to execute')
     ])
    ])

	if(params.BUILD_MECHANISM == 'BUILD') {
		stage 'SourceCodeBuild'	
			UDF_BuildSourceCode()
			
		stage 'SonarQube'
			UDF_ExecuteSonarQubeRules()			
	} else if(params.BUILD_MECHANISM == 'BUILD-AND-RELEASE') {
		try{
			stage 'SourceCodeBuild'	
				UDF_BuildSourceCode()
				
			stage 'SonarQube'
				UDF_ExecuteSonarQubeRules()
		
			stage 'ArtifactUploadToNexus'
				UDF_ArtifactUploadToNexus()
				
			stage 'DeployToCloudHub'
				UDF_DeployToCloudHub()
			
			stage 'Notification'
				SendEmail("","","success")			
		} catch(error) {
			throw(error)
			SendEmail("","","Failed")
		}
	} else if(params.BUILD_MECHANISM == 'BUILD-AND-ARTEFACT-UPLOAD') {
		try{
			stage 'SourceCodeBuild'	
				UDF_BuildSourceCode()
				
			stage 'SonarQube'
				UDF_ExecuteSonarQubeRules()
				
			stage 'ArtifactUploadToNexus'
				UDF_ArtifactUploadToNexus()
			
		} catch(error) {
			throw(error)
			SendEmail("","","Failed")
		}
	} else if(params.BUILD_MECHANISM == 'RELEASE') {
		try{
		
		stage 'GetArtifactListFromNexus'					
		def downloadDir = ""

		withCredentials([usernamePassword(credentialsId: 'bcbacb84-8abf-482f-be12-4bc25148b805',passwordVariable: 'nexuspassword',usernameVariable: 'nexususername')])  {
		def nexus_SearchURL = "curl -v -u ${nexususername}:${nexuspassword} \"${nexus_BaseURL}/service/rest/beta/search?repository=${nexus_RepoName}&group=${pom_GroupID}&name=${pom_ArtifactId}\""

		def nexusVersionInput = input(
            		id: 'nexusVersionInput', message: 'Please select Nexus Artifact Version for Deployment:?', 
            		parameters: [
					[
						$class: 'ChoiceParameterDefinition', choices: UDF_GetNexusArtifactsList(nexus_SearchURL), 
						name: 'SELECTED_NEXUS_VERSION',
						description: 'A select box option'
		]])

			String selectedVersion = "${nexusVersionInput}"
			selectedVersion = selectedVersion.split(":")[1]	
			
			def nexus_SearchURL_File = "curl -v -u ${nexususername}:${nexuspassword} \"${nexus_BaseURL}/service/rest/beta/search?repository=${nexus_RepoName}&group=${pom_GroupID}&name=${pom_ArtifactId}&version=${selectedVersion}\""
			downloadDir = UDF_GetNexusArtifacts_DownloadURL(nexus_SearchURL_File)
			}
			
			def DomainNameUserInput = input(
			 id: 'DomainNameUserInput', message: 'Enter app/domain name for CloudHub Deployment:?', 
			 parameters: [
			 [$class: 'TextParameterDefinition', defaultValue: '', description: 'CloudHub Domain Name', name: 'DomainName']
			])
			
			def propertiesFilePath = "${env.JENKINS_HOME}\\CloudHub\\"+UDF_GetGitRepoName()+"\\${params.ENVIRONMENTS}.properties.txt"
			
			//def downloadDir = "${env.JENKINS_HOME}/CloudHub/Downloads/"+UDF_GetGitRepoName()		
			
			/*withCredentials([usernamePassword(
			credentialsId: 'bcbacb84-8abf-482f-be12-4bc25148b805',
			passwordVariable: 'nexuspassword',
			usernameVariable: 'nexususername')])
			{
			def nexus_SearchURL_File = "curl -v -u ${nexususername}:${nexuspassword} ${nexus_BaseURL}/service/rest/beta/search?repository=${nexus_RepoName}&group=${pom_GroupID}&name=${pom_ArtifactId}&version=${selectedVersion}"
			def downloadDir = UDF_GetNexusArtifacts_DownloadURL(nexus_SearchURL_File)
			}*/
			//def downloadFilePath="\${env.JENKINS_HOME}\\CloudHub\\Downloads\\"+UDF_GetGitRepoName()+"\\${pom_ArtifactId}.jar"
			def downloadFilePath="C:\\Program Files\\Jenkins\\CloudHub\\Downloads\\"+UDF_GetGitRepoName()+"\\${pom_ArtifactId}.jar"
			//def downloadFilePath="${env.WORKSPACE}/target/${pom_ArtifactId}-${pom_Version}.zip"

		stage 'DeployToCloudHub'
				UDF_DeployToCloudHub(downloadFilePath, propertiesFilePath,downloadDir,DomainNameUserInput)
			
		stage 'Notification'
			SendEmail("","","success")
		
		}catch(error) {
			//SendEmail()
			throw(error)
			SendEmail("","","Failed")
		}
	}
}

//BUILD STAGE
def UDF_BuildSourceCode()
{	
	try	{
		if (params.BuildParameters == '')
		{
	echo 'Build is Starting'
	//sh 'mvn -U install -DskipTests=true'
	sh 'mvn -version'
	echo 'Build Completed'	
		}else
		{
		echo 'Build is Starting'
		sh '${BuildParameters}'
		echo 'Build Completed'	
		}
	}catch(error) {
		throw(error)
		SendEmail("","","Failed")
	}
}

/*
SONARQUBE - STAGE
This function provides functionality to do the SONAR Analysis
*/
def UDF_ExecuteSonarQubeRules()
{	
	try{
		echo 'SonarQube Rules Execution started'
		withSonarQubeEnv('SonarServer-Local') {
			//sh 'mvn sonar:sonar'
		}
		echo 'SonarQube Rules Execution Completed'	
	} catch(error) {
		throw(error)
		SendEmail("","","Failed")
	}
}

/*
NEXUS ARTIFACT UPLOAD - STAGE
This function provides functionality to upload the artifacts to the Nexus repository
*/

def UDF_ArtifactUploadToNexus() 
{	
	try{
		echo "Artifact Copy to Nexus Started"
		def nexus_Protocol = "http"
		def nexus_BaseURL = "${env.LOCAL_NEXUS_BASEURL}"		
		def nexus_RepoName = UDF_Get_Nexus_RepoName("${params.ENVIRONMENTS}")		
		def pom_GroupID = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","groupId")
		pom_ArtifactId = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","artifactId")
		def pom_Version = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","version")
		def pom_Packaging = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","packaging")
		//def nexus_SearchURL = "${nexus_BaseURL}/service/rest/beta/search?repository=${nexus_RepoName}&group=${pom_GroupID}&name=${pom_ArtifactId}"	
		def downloadDir = "${env.JENKINS_HOME}\\CloudHub\\Downloads\\"+UDF_GetGitRepoName()	
		
		echo "###### NEXUS REPO DETAILS ######"
		echo "Nexus base URL: ${env.LOCAL_NEXUS_BASEURL}"	
		echo "nexus_RepoName: ${params.ENVIRONMENTS}"
		echo "pom_GroupID: ${pom_GroupID}"
		echo "pom_ArtifactId is : ${pom_ArtifactId}"
		echo "pom_Version is : ${pom_Version}"
		echo "pom_Packaging is : ${pom_Packaging}"
		echo "downloadDir is : ${downloadDir}"
	
	String nexusRepoName = "${nexus_RepoName}/"
	String targetZipName = "target/${pom_ArtifactId}-${pom_Version}-mule-application.jar"

	echo "nexusRepoName is : ${nexusRepoName}"
	echo "targetZipName is : ${targetZipName}"
	
	if(nexus_BaseURL.contains("http://")) {
		nexus_BaseURL = nexus_BaseURL.substring(7)
	} else if(nexus_BaseURL.contains("https://")) {
		nexus_BaseURL = nexus_BaseURL.substring(8)
	}
	echo "nexus_BaseURL after substring is : ${nexus_BaseURL}"

	nexusArtifactUploader(
		nexusVersion: 'nexus3',
		protocol: nexus_Protocol,
		nexusUrl: nexus_BaseURL,
		groupId: pom_GroupID,
		version: pom_Version,
		repository: nexusRepoName,
		credentialsId: 'bcbacb84-8abf-482f-be12-4bc25148b805',
		artifacts: [
			[artifactId: pom_ArtifactId,
			 classifier: '',
			 file: targetZipName,
			 type: 'jar']
		]
	 )
    echo 'Artifact Copy to Nexus Completed'
	}catch(error) {
		throw(error)
		SendEmail("","","Failed")
	}
}

/*
DEPLOY STAGE
This function provides functionality to deploy the application package(zip file) to CloudHub Runtime
*/

def UDF_DeployToCloudHub()
{	

	echo "###### Entered to Application Deployment stage ######"

	def DomainNameUserInput = input(
		 id: 'DomainNameUserInput', message: 'Enter app/domain name for CloudHub Deployment:?', 
		 parameters: [
		 [$class: 'TextParameterDefinition', defaultValue: '', description: 'CloudHub Domain Name', name: 'DomainName']
	])

	def propertiesFilePath = "${env.JENKINS_HOME}\\CloudHub\\"+UDF_GetGitRepoName()+"\\${params.ENVIRONMENTS}.properties.txt"
	def downloadFilePath="${env.WORKSPACE}\\target\\${pom_ArtifactId}-${pom_Version}-${pom_Packaging}.jar"
	
	echo "propertiesFilePath is : ${propertiesFilePath}"
	echo "downloadFilePath is : ${downloadFilePath}"	
	echo "DomainName which you have entered is: ${DomainNameUserInput}"
	echo "APP_EXISTS is : ${APP_EXISTS}"

	def AnypointCredentialID = input(
			 id: 'DomainNameUserInput', message: 'Please provide Cloudhub Credential ID:?', 
			 parameters: [
			 [$class: 'TextParameterDefinition', defaultValue: '', description: 'Cloudhub Credential', name: 'DomainName']
			])
			
	def PropertiesFileInput = input(
                id: 'PropertiesFileInput', message: 'Is Properties file required for the deployment:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: 'YES\nNO', 
					name: 'SELECTED_PROPERTY_FILE',
					description: 'Please confirm YES or NO'
				]])
		
	def AnypointOrganization = input(
                id: 'AnypointOrganization', message: 'Please select CloudHub Anypoint Organization name to deploy', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: 'Oasis Fashions\nLnD', 
					name: 'ANYPOINT_ORGANIZATION',
					description: 'Please select the Organization'
				]])
	def AnypointEnvironment = input(
			 id: 'AnypointEnvironment', message: 'Please provide Cloudhub Anypoint Environment name to deploy', 
			 parameters: [
			 [$class: 'TextParameterDefinition', defaultValue: 'Example: DEV', description: 'Cloudhub Environment Name', name: 'DomainName']
			])
		
	environment {
		ANYPOINT_CREDENTIALS = credentials("${AnypointCredentialID}")
	}
	
	echo "Nexus Download URL - ${udfp_NexusPackageURL} ### Download file path - ${udfp_DownloadedFilePath}"
	
	if(udfp_NexusPackageURL != "")
	{
		withCredentials([usernamePassword(credentialsId: 'bcbacb84-8abf-482f-be12-4bc25148b805',passwordVariable: 'nexuspassword',usernameVariable: 'nexususername')]) {			
			sh "wget --user ${nexususername} --password ${nexuspassword} ${udfp_NexusPackageURL} -O \"${udfp_DownloadedFilePath}\""	
		}
	}
	
	def appExists = false
	try{		
		echo 'Auto-deploy to CloudHub started'
		echo "AnypointCredentialID: ${AnypointCredentialID}"
		echo "ANYPOINT_USERNAME : ${username1}"
		echo "ANYPOINT_PASSWORD: ${password1}"
		withCredentials([usernamePassword(credentialsId: "${AnypointCredentialID}",passwordVariable: 'password1',usernameVariable: 'username1')]) {
			sh """			
				export ANYPOINT_USERNAME=${username1}
				export ANYPOINT_PASSWORD=${password1}
				export ANYPOINT_ORG="${AnypointOrganization}"
				export ANYPOINT_ENV="${AnypointEnvironment}"
				anypoint-cli runtime-mgr cloudhub-application describe ${udfp_AppName}
			"""
		}
		appExists = true
	} catch(err) {
		//throw(err)
		//SendEmail("","","FAILED")
	}
	
	if(appExists == false && PropertiesFileInput == 'YES') {
		echo "App Exists is false and Property file input is YES"
		
		def workerSizeInput = input(
                id: 'workerSizeInput', message: 'Please select vCores for Deployment:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: '0.1\n0.2\n1\n2\n4\n8\n16', 
					name: 'SELECTED_WORKER_VERSION',
					description: 'Please select worker version'
				]])
		def workerNumberInput = input(
                id: 'workerInput', message: 'Please select workerSize for Deployment:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: '1\n2\n3\n4\n5\n6\n7\n8', 
					name: 'SELECTED_WORKERS',
					description: 'Please select No.of workers'
				]])
		def RuntimeInput = input(
                id: 'workerInput', message: 'Please select mule runtime for Deployment:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: '4.1.3\n4.1.4\n4.1.5', 
					name: 'SELECTED_MULE_RUNTIME',
					description: 'Please select Mule Runtime'
				]])
				
		vCoreInput = "${workerSizeInput}"
		workerInput = "${workerNumberInput}"
		runTimeVersion = "${RuntimeInput}"
				
		echo "###### Deploy New Application ######"
		withCredentials([usernamePassword(
			credentialsId: "${AnypointCredentialID}",
			passwordVariable: 'password1',
			usernameVariable: 'username1')])
			{
			sh """			
			export ANYPOINT_USERNAME=${username1}
			export ANYPOINT_PASSWORD=${password1}
			export ANYPOINT_ORG="${AnypointOrganization}"
			export ANYPOINT_ENV="${AnypointEnvironment}"
			anypoint-cli runtime-mgr cloudhub-application deploy ${udfp_AppName} \"${udfp_DownloadedFilePath}\" --workerSize ${vCoreInput} --workers ${workerInput} --runtime ${runTimeVersion} --propertiesFile ${udfp_PropertiesFilePath}
		"""
			}
		echo "###### New Application Deployment is done ######"
		
	}
	else if(appExists == false && PropertiesFileInput == 'NO')
	{
		//Modify Existing Application
		echo "App Exists is false and Property file input is NO, Hence Modify Existing Application"
		def workerSizeInput = input(
                id: 'workerSizeInput', message: 'Please select vCores for Deployment:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: '0.1\n0.2\n1\n2\n4\n8\n16', 
					name: 'SELECTED_WORKER_VERSION',
					description: 'Please select worker version'
				]])
		def workerNumberInput = input(
                id: 'workerInput', message: 'Please select workerSize for Deployment:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: '1\n2\n3\n4\n5\n6\n7\n8', 
					name: 'SELECTED_WORKERS',
					description: 'Please select No.of workers'
				]])
		def RuntimeInput = input(
                id: 'workerInput', message: 'Please select mule runtime for Deployment:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: '4.1.3\n4.1.4\n4.1.5', 
					name: 'SELECTED_MULE_RUNTIME',
					description: 'Please select Mule Runtime'
				]])
				
		vCoreInput = "${workerSizeInput}"
		workerInput = "${workerNumberInput}"
		runTimeVersion = "${RuntimeInput}"

		echo "###### Displaying all the parameters before deploying to cloud hub #####"
		echo "appExists is : ${appExists}"
		echo "PropertiesFileInput is : ${PropertiesFileInput}"		
		echo "vCoreInput is : ${vCoreInput}"
		echo "workerInput is : ${workerInput}"
		echo "runTimeVersion is : ${runTimeVersion}"
		echo "AppName is : ${udfp_AppName}"
		echo "Download path is : ${udfp_DownloadedFilePath}"
		
		withCredentials([usernamePassword(
			credentialsId: "${AnypointCredentialID}",
			passwordVariable: 'password1',
			usernameVariable: 'username1')])
			{
			sh """			
			export ANYPOINT_USERNAME=${username1}
			export ANYPOINT_PASSWORD=${password1}
			export ANYPOINT_ORG="${AnypointOrganization}"
			export ANYPOINT_ENV="${AnypointEnvironment}"
			anypoint-cli runtime-mgr cloudhub-application deploy ${udfp_AppName} \"${udfp_DownloadedFilePath}\" --workerSize ${vCoreInput} --workers ${workerInput} --runtime ${runTimeVersion}
		"""
			}
		echo 'new app and no props'
	} else if (appExists == true && PropertiesFileInput == 'YES') {
	
		//Modify Existing Application
		echo "App Exists is true and Property file input is YES, Hence Modify Existing Application"

		def MuleRunTImeProperties = input(
                id: 'MuleRunTImeProperties', message: 'appExists == true && PropertiesFileInput ==  NO:Do you want to change Mule Runtime like vCore,Workers, MuleRuntimeVersion:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: 'YES\nNO', 
					name: 'MULE_RUNTIME_PROPERTIES',
					description: 'appExists == true && PropertiesFileInput ==  NO:Please confirm YES or NO'
				]])
		if (MuleRunTImeProperties == 'YES')
		{
			def workerSizeInput = input(
                id: 'workerSizeInput', message: 'appExists == true && PropertiesFileInput ==  NO:Please select vCores for Deployment:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: '0.1\n0.2\n1\n2\n4\n8\n16', 
					name: 'SELECTED_WORKER_VERSION',
					description: 'appExists == true && PropertiesFileInput ==  NO:Please select worker version'
				]])
		def workerNumberInput = input(
                id: 'workerInput', message: 'appExists == true && PropertiesFileInput ==  NO:Please select workerSize for Deployment:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: '1\n2\n3\n4\n5\n6\n7\n8', 
					name: 'SELECTED_WORKERS',
					description: 'appExists == true && PropertiesFileInput ==  NO:Please select No.of workers'
				]])
		def RuntimeInput = input(
                id: 'workerInput', message: 'appExists == true && PropertiesFileInput ==  NO:Please select mule runtime for Deployment:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: '4.1.3\n4.1.4\n4.1.5\n4.2.0', 
					name: 'SELECTED_MULE_RUNTIME',
					description: 'appExists == true && PropertiesFileInput ==  NO:Please select Mule Runtime'
				]])
				
		vCoreInput = "${workerSizeInput}"
		workerInput = "${workerNumberInput}"
		runTimeVersion = "${RuntimeInput}"
		
		echo "udfp_AppName is: ${udfp_AppName}"
		echo "udfp_DownloadedFilePath is: ${udfp_DownloadedFilePath}"
		echo "vCoreInput: ${vCoreInput}"
		echo "workerInput: ${workerInput}"
		echo "runTimeVersion: ${runTimeVersion}"
		echo "udfp_PropertiesFilePath: ${udfp_PropertiesFilePath}"
		
		withCredentials([usernamePassword(credentialsId: "${AnypointCredentialID}",passwordVariable: 'password1',usernameVariable: 'username1')]) {
			sh """			
			export ANYPOINT_USERNAME=${username1}
			export ANYPOINT_PASSWORD=${password1}
			export ANYPOINT_ORG="${AnypointOrganization}"
			export ANYPOINT_ENV="${AnypointEnvironment}"
			//anypoint-cli runtime-mgr cloudhub-application modify ${udfp_AppName} \"${udfp_DownloadedFilePath}\" --workerSize ${vCoreInput} --workers ${workerInput} --runtime ${runTimeVersion} --propertiesFile ${udfp_PropertiesFilePath}
			mvn deploy -DmuleDeploy -Dmule.env=dev -Dencrypt.key=MULESOFT_DEV -Danypoint.platform.gatekeeper=disabled
		"""
		}
		echo 'existing app and props'
		}else{
			echo "Application exists but No Property file"
			echo "AnypointCredentialID: ${AnypointCredentialID}"
			
			withCredentials([usernamePassword(
			credentialsId: "${AnypointCredentialID}",
			passwordVariable: 'password1',
			usernameVariable: 'username1')])
			{
			sh """			
			export ANYPOINT_USERNAME=${username1}
			export ANYPOINT_PASSWORD=${password1}
			export ANYPOINT_ORG="${AnypointOrganization}"
			export ANYPOINT_ENV="${AnypointEnvironment}"
			anypoint-cli runtime-mgr cloudhub-application modify ${udfp_AppName} \"${udfp_DownloadedFilePath}\"  --propertiesFile ${udfp_PropertiesFilePath}
		"""
			}
		}
	} else if (appExists == true && PropertiesFileInput == 'NO') {
	
		//Modify Existing Application
		echo "App Exists is true and Property file input is NO, Hence Modify Existing Application"
	
		def MuleRunTImeProperties = input(
                id: 'MuleRunTImeProperties', message: 'Do you want to change Mule Runtime like vCore,Workers, MuleRuntimeVersion:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: 'YES\nNO', 
					name: 'MULE_RUNTIME_PROPERTIES',
					description: 'Please confirm YES or NO'
				]])
				
		if (MuleRunTImeProperties == 'YES')
		{
			def workerSizeInput = input(
                id: 'workerSizeInput', message: 'Please select vCores for Deployment:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: '0.1\n0.2\n1\n2\n4\n8\n16', 
					name: 'SELECTED_WORKER_VERSION',
					description: 'Please select worker version'
				]])
		def workerNumberInput = input(
                id: 'workerInput', message: 'Please select workerSize for Deployment:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: '1\n1\n3\n4\n5\n6\n7\n8', 
					name: 'SELECTED_WORKERS',
					description: 'Please select No.of workers'
				]])
		def RuntimeInput = input(
                id: 'workerInput', message: 'Please select mule runtime for Deployment:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: '4.1.3', 
					name: 'SELECTED_MULE_RUNTIME',
					description: 'Please select Mule Runtime'
				]])
				
		vCoreInput = "${workerSizeInput}"
		workerInput = "${workerNumberInput}"
		runTimeVersion = "${RuntimeInput}"
		/*environment {
        ANYPOINT_CREDENTIALS = credentials("${AnypointCredentialID}")
		}*/
		echo '${AnypointOrganization}'
		withCredentials([usernamePassword(
			credentialsId: "${AnypointCredentialID}",
			passwordVariable: 'password1',
			usernameVariable: 'username1')])
			{
		
		sh """			
			export ANYPOINT_USERNAME=${username1}
			export ANYPOINT_PASSWORD=${password1}
			export ANYPOINT_ORG="${AnypointOrganization}"
			export ANYPOINT_ENV="${AnypointEnvironment}"
			anypoint-cli runtime-mgr cloudhub-application modify ${udfp_AppName} \"${udfp_DownloadedFilePath}\"  --workerSize ${vCoreInput} --workers ${workerInput} --runtime ${runTimeVersion} 
		"""
			}
		echo 'existing app and props'
		}
		else{
		withCredentials([usernamePassword(
			credentialsId: "${AnypointCredentialID}",
			passwordVariable: 'password1',
			usernameVariable: 'username1')])
			{
			sh """			
			export ANYPOINT_USERNAME=${username1}
			export ANYPOINT_PASSWORD=${password1}
			export ANYPOINT_ORG="${AnypointOrganization}"
			export ANYPOINT_ENV="${AnypointEnvironment}"
			anypoint-cli runtime-mgr cloudhub-application modify ${udfp_AppName} \"${udfp_DownloadedFilePath}\"
		"""
			}
		}
	}
}

def UDF_GetGitRepoName()
{	
	try{
	def tokens = "${env.JOB_NAME}".tokenize('/')
    String repo = tokens[tokens.size()-2]
	
    //branch = tokens[tokens.size()-1]	
    //org = tokens[tokens.size()-3]	
	
	return repo
	}catch(error)
	{
		throw(error)
		SendEmail("","","Failed")
	}
}

def UDF_Get_Nexus_RepoName(udfp_Environment)
{	
	try{
	echo "Start of UDF_Get_Nexus_RepoName"
	def repoName = ""
	
	if(udfp_Environment == "DEV" || udfp_Environment == "SIT" || udfp_Environment == "UAT")
	{		
		repoName = "${env.LOCAL_NEXUS_NONPROD_REPONAME}"
	}
	else if(udfp_Environment == "PROD")
	{		
		repoName = "${env.LOCAL_NEXUS_PROD_REPONAME}"
	}
	echo "End of UDF_Get_Nexus_RepoName"
	
	return repoName
	}catch(error)
	{
		throw(error)
		SendEmail("","","Failed")
	}
}

def UDF_GetPOMData(udfp_PomName, udfp_PropertyName)
{	
	try{
	def resultVal = ""
	def pomFile = readFile(udfp_PomName)
	def pom = new XmlParser().parseText(pomFile)
	def gavMap = [:]
	resultVal =  pom[udfp_PropertyName].text().trim()
	return resultVal
	}catch(error)
	{
		throw(error)
		SendEmail("","","Failed")
	}
}

/*
This function returns the list of Packages from given Nexus Repository
*/

def UDF_GetNexusArtifactsList(udfp_NexusPackageSearchURL)
{	
	try{
	String responseString = ""
	//String nexusResponseJson = udfp_NexusPackageSearchURL.toURL().text
	def nexusResponseJson = sh(returnStdout: true, script: udfp_NexusPackageSearchURL).trim()
	
	//def nexusResponseJson = sh udfp_NexusPackageSearchURL
	
	//echo "After function call - Output:${nexusResponseJson}"		
	
	def nexusResponseJsonMap = parseJsonToMap(nexusResponseJson)
	
	for (int i = 0; i < nexusResponseJsonMap.items.size(); i++) {
		responseString = responseString+ nexusResponseJsonMap.items[i].name+":"+nexusResponseJsonMap.items[i].version+"\n"
	}
	
	return responseString
	}catch(error)
	{
		throw(error)
		SendEmail("","","Failed")
	}
}

def UDF_GetNexusArtifacts_DownloadURL(udfp_NexusPackageSearchURL)
{	
	try{
	String responseString = ""
	//String nexusResponseJson = udfp_NexusPackageSearchURL.toURL().text
	
	def nexusResponseJson = sh(returnStdout: true, script: udfp_NexusPackageSearchURL).trim()
	
	echo "Function call - Output:${nexusResponseJson}"		
	
	def nexusResponseJsonMap = parseJsonToMap(nexusResponseJson)
	
	if(nexusResponseJsonMap.items.size() > 0)
	{			
		responseString = nexusResponseJsonMap.items[0].assets[0].downloadUrl
	}
	
	return responseString
	}catch(error)
	{
		throw(error)
		SendEmail("","","Failed")
	}
}

/*
This function Sends Email
*/
def SendEmail(udfp_ToAddress, udfp_FromAddress, udfp_Status)
{	
	try{
	   String body = ""
	   
	   if(udfp_Status == "success")
	   {
		   body= "SUCCESS"
	   }
	   else
	   {
		   body= "FAILED"
	   }
		
		mail subject: "${env.JOB_NAME} (${env.BUILD_NUMBER}) ${body}",
				body: "It appears that ${env.BUILD_URL} is ${body}",
				  to: "michal.giela@oasis-warehouse.com",
			 replyTo: "webmaster@oasis-stores.com",
				from: "Jenkins@oasis-stores.com"
				
	}catch(error)
	{		
		throw(error)
	}
}

/*
This function converts the given json string to HashMap and returns it back
*/

import groovy.json.JsonSlurperClassic
@NonCPS
def parseJsonToMap(String json) {
    final slurper = new JsonSlurperClassic()
    return new HashMap<>(slurper.parseText(json))
}
