def myVariable = "C:/ProgramData/jenkins/.jenkins/workspace/PiplineSCM/aspnet-core-dotnet-core/aspnet-core-dotnet-core.csproj"
pipeline 
{
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
                        bat "\"${scannerHome}\\SonarScanner.MSBuild.exe\" begin /k:\"DotNetApp\""
                    
                        bat "dotnet build ${myVariable}"
                        bat "\"${scannerHome}\\SonarScanner.MSBuild.exe\" end"
                    }
                }
            }
        }
        /*stage("Quality Gate") 
        {
            steps
              {
                waitForQualityGate abortPipeline: true
              } 
        }*/
        stage('build')
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
        stage('publish')
        {
            steps
            {
                bat "dotnet publish ${myVariable}"
            }
        }
        stage('Package') 
        {
            steps 
                {
                echo "Deploying to stage environment for more tests!";
                bat "del *.zip"
        
               // bat "tar.exe -a -c -f sampleapp.zip ${myVariable} "
			bat "tar.exe -a -c -f sampleapp_${BUILD_NUMBER}.zip ${myVariable} "	
      
                }
        }
        stage ('Connected to jfrog')
        {
            steps
            {
               rtServer (
                 id: "Artifactory",
                 url: 'http://localhost:8082/artifactory',
                 //username: 'admin',
                  //password: 'Aditya@1998',
                  bypassProxy: true,
                   timeout: 300
                        )
            }
        }
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
        }
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
        }
         stage('Expand the Archive') 
	{
	   steps
	   {
		   powershell '''
		                  Expand-Archive 'F:/jfrog/sampleapp.zip' -DestinationPath 'F:/zipfile/'
		              '''
            } 
	}
	   stage('Deploy') 
	{
	   steps
		{
			azureWebAppPublish appName: "${env.appName}", azureCredentialsId: 'Azure', resourceGroup: "${env.resourceGroup}"
	         }
	}
        
        
	}
}
