// Defining Variable
def myVariable = "C:/ProgramData/jenkins/.jenkins/workspace/PiplineSCM/aspnet-core-dotnet-core/aspnet-core-dotnet-core.csproj"
pipeline 
{
	//Define Environment Variable regarding azure webApp
	environment
	{
		appName = "Aditya-Jenkin-App"
		resourceGroup = "Training-rg"
	}	
    agent any
	stages 
	{
	    /*stage('Git CheckOut') {
		    steps {
			   git branch: 'main', url: 'https://github.com/AdityaRai1998/sampleapp-1.git'
			}
		}*/
		//Static code analysis with Sonarqube
		stage('SonarQube Analysis') 
        {
            steps
            {
                script
                {
                    def msbuildHome = tool 'MSBuild'        
                    def scannerHome = tool 'SonarScanner for MSBuild'
                    withSonarQubeEnv('SonarQubeServer') 
                    {
                        bat "\"${scannerHome}\\SonarScanner.MSBuild.exe\" begin /k:\"WebApp\""
                    
                        bat "dotnet build ${myVariable}"
                        bat "\"${scannerHome}\\SonarScanner.MSBuild.exe\" end"
                    }
                }
            }
        }
        stage("Quality gate") 
        {
            steps {
               waitForQualityGate abortPipeline: false, credentialsId: 'aditya'
            }
        }
        stage('build') // Building the Application
        {
            steps
            {
                bat "dotnet build ${myVariable} --configuration Release"
                //C:\ProgramData\Jenkins\.jenkins\workspace\demo1\aspnet-core-dotnet-core
            }
        }
        stage('Test')
        {
            steps
            {
                bat "dotnet test ${myVariable}"
                //C:\ProgramData\Jenkins\.jenkins\workspace\demo1\aspnet-core-dotnet-core
            }
        }
        stage('publish') // Publishing the webapp
        {
            steps
            {
                bat "dotnet publish ${myVariable}"
            }
        }
		//Packaging with  Latest build number
        stage('Package') 
        {
            steps 
                {
                echo "Deploying to stage environment for more tests!";
                bat "del *.zip" // Deleting our previsioly created file
        
               // bat "tar.exe -a -c -f sampleapp.zip ${myVariable} "
			bat "tar.exe -a -c -f sampleapp_${BUILD_NUMBER}.zip ${myVariable} "	
      
                }
        }
		//This Stage is for connecting Jfrog with jenkins
        stage ('Connected to jfrog')
        {
            steps
            {
               rtServer (
                 id: "Artifactory",
                 //url: 'http://localhost:8082/artifactory',
                 //username: 'admin',
                  //password: 'Aditya@1998',
                  bypassProxy: true,
                   timeout: 300
                        )
            }
        }       // Uploading file to jfrog repository in zip pattern
                stage('Upload file to jfrog'){
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
        } //Download artifact from jfrog repository to our local system
        stage ('download the artifacts from artifactory')
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
        } // Expand the downloded file to destination folder
         stage('Expand the Archive') 
	{
	   steps
	   {
		   powershell '''
		                  Expand-Archive 'F:/jfrog/sampleapp.zip' -DestinationPath 'F:/zipfile/'
		              '''
            } 
	}
		// Deploy our Web App on Azure Cloud
	   stage('Deploy') 
	{
	   steps
		{
			azureWebAppPublish appName: "${env.appName}", azureCredentialsId: 'Azure', resourceGroup: "${env.resourceGroup}"
	         }
	}
        
        
	}
}
