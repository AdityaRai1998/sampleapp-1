                                                                                                                              
def myVariable = "C:/ProgramData/jenkins/.jenkins/workspace/PiplineSCM/aspnet-core-dotnet-core/aspnet-core-dotnet-core.csproj"        // Defining Variable
pipeline 
{
	
	environment                                            // Define Environment Variable regarding azure webApp
	{
		appName = "Aditya-Jenkin-App"                  // WebApp name created on Azure Platform
		resourceGroup = "Training-rg"                  // Resource Group Created on Azure Platform
	}	
    agent any                                                  // Pipeline starts with available agents
	stages 
	{
		stage('SonarQube Analysis')                 // Static Code Analysis on Soanrqube Server
        {
            steps
            {
                script
                {
                    def msbuildHome = tool 'MSBuild'                                       // MSBuild is the name of MSBuild plugin on jenkins       
                    def scannerHome = tool 'SonarScanner for MSBuild'                      // 'SonarScanner for MSBuild' is the name of Sonar Scanner Plugin on jenkins
                    withSonarQubeEnv('SonarQubeServer')                                    // 'SonarQubeServer' is the name of sonarqube server plugin on jenkins
                    {
                        bat "\"${scannerHome}\\SonarScanner.MSBuild.exe\" begin /k:\"WebApp\""      // Here We need to define Project name creted on sonarqube server
                    
                        bat "dotnet build ${myVariable}"                                            // Run the command dotnet build for building the application
                        bat "\"${scannerHome}\\SonarScanner.MSBuild.exe\" end"                      // Sonar Scanner for msbuild plugin will perform building operation
                    }
                }
            }
        }
        stage("Quality gate")                                                        // A quality gate is a milestone in an IT project that requires that predefined criteria be met before the project can proceed to the next phase. 
        {
            steps {
               waitForQualityGate abortPipeline: true, credentialsId: 'aditya'     
            }
        }
        stage('build')                                                                 // Building the Application
        {
            steps
            {
                bat "dotnet build ${myVariable} --configuration Release"              // command is dotnet build for build the appliaction
            }
        }
        stage('Test')                                                   // Perform Unit test
        {
            steps
            {
                bat "dotnet test ${myVariable}"
            }
        }
        stage('publish')                                        // Publishing the webapp
        {
            steps
            {
                bat "dotnet publish ${myVariable}"             // Publishing our weeb App Solution on target location
            }
        }
        stage('Package')                                      //Packaging with  Latest build number
        {
            steps 
                {
                echo "Deploying to stage environment for more tests!";
                bat "del *.zip"                                                   // Deleting our previsioly created Zip file
        
               // bat "tar.exe -a -c -f sampleapp.zip ${myVariable} "
			bat "tar.exe -a -c -f sampleapp_${BUILD_NUMBER}.zip ${myVariable} "	 // Packaging our file with name & latest build
      
                }
        }
        stage ('Connected to jfrog')                   //This Stage is for connecting Jfrog with jenkins
        {
            steps
            {
               rtServer (
                 id: "Artifactory",       // This is the id have creted on jenkins in configuration tool with all credentials of Jfrog Artifactory
                  bypassProxy: true,
                   timeout: 300
                        )
            }
        }   
                stage('Upload file to jfrog'){               // Uploading file to jfrog repository in zip pattern on target had created on JFrog  local Repository
            steps{
                rtUpload (
                 serverId:"Artifactory" ,
                  spec: '''{
                   "files": [
                      {
                      "pattern": "*.zip",
                      "target": "aditya-webapp-jenkin-libs-release"        
                      }
                            ]
                           }''',
                        )
            }
        }
        stage ('Publish build info') 
        {
            steps 
            {
                rtPublishBuildInfo (
                    serverId: "Artifactory"
                )
            }
        } 
        stage ('download the artifacts from artifactory')             //Download artifact from jfrog repository to our local system
        {
            steps
            {
                  rtDownload (
                    serverId: "Artifactory",
                        spec:
                              """{
                                "files": [
                                  {
                                    "pattern": "aditya-webapp-jenkin-libs-release/*.zip",
                                    "target": "F:/jfrog/"          
                                  }
                               ]
                              }"""
      )
            }
        } 
        
	   stage('Deploy')           		// Deploy our Web App on Azure Cloud
	{
	   steps
		{
			azureWebAppPublish appName: "${env.appName}", azureCredentialsId: 'Azure', resourceGroup: "${env.resourceGroup}"    // All Variable defined in Envoronment variable section & calling with env.(variable name)
	         }
	}
        
        
	}
}
