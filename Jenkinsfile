	def ReposBaseURL = "https://github.com/kenjioshio"
	def PackageName = "Calculation"
	def TargetRepos = "${ReposBaseURL}/${PackageName}.git"
	def BranchName = "master"
	def WSDir = "${env.CI_BASE}\\${BranchName}\\${PackageName}\\${env.BUILD_TIMESTAMP}"
	def MSBUILD_EXE ="${env.MSBUILD40}"
	def BUILD_OPTIONS = "/p:Platform=x86 /p:DebugSymbols=true /p:DebugType=pdbonly"
	
pipeline
{

    agent none
    stages
    {
        stage('Checkout') 
        {
            agent
            {
                label "master"
            }
            steps
            {
                ws("${WSDir}\\Checkout/")
                {
                    checkout([$class: 'GitSCM', branches: [[name: '*/$BranchName']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'b6f74b6b-1f9f-4da8-b596-ab968bee00a7', url: '$TargetRepos']]])
                }
            }        
        }
        stage('Build') 
        {
            agent
            {
                label "master"
            }
            steps
            {
                ws("${WSDir}\\Checkout/")
                {
                    script
                    {
                        powershell '.\\.nuget\\nuget.exe restore Calculation.sln -configfile .\\.nuget\\NuGet.Config'
                        bat '%MSBUILD_EXE% .\\%PackageName%.sln /t:rebuild %BUILD_OPTIONS%' 
                        bat '%ChocoToolBase%\\7z.exe a %WSDir%\\Checkout\\%PackageName%_%BUILD_TIMESTAMP%.zip %WSDir%\\Checkout\\bin\\*'
                        archiveArtifacts '*.zip'
                    }
                }
            }
        }
        stage('Static Analyis') 
        {
            parallel
            {
                stage('SA-CLOC')
                {
                    agent
                    {
                        label "master"
                    }
                    steps
                    {
                        ws("${WSDir}\\cloc/")
                        {
                            bat '%ChocoToolBase%\\cloc.exe ..\\Checkout --csv --out=.\\cloc_result.csv'
                            archiveArtifacts 'cloc_result.csv'
                        }        
                    }
                }
                stage('Protex')
                {
                    agent
                    {
                        label "master"
                    }
                    steps
                    {
                        ws("${WSDir}\\Protex/")
                        {
                            powershell 'write-host "Here we are!!"'
                        }        
                    }
                }
            }
        }
        stage('UnitTest') 
        {
            agent
            {
                label "master"
            }
            steps
            {
                ws("${WSDir}\\UT/")
                {
                    script
                    {
                        try
                        {
                            bat '''REM Unit testing is executed though Coverage Tool
                            Echo Setting Unit Testing Precondition
                            REM TEST_WS:A location of Test assembly.  
                            set TEST_WS =..\\checkout\\bin_test\\
                            set TARGET_TEST_DLL=SummationTest.dll
                            set FILTERS=+[Summation]* 
                            set FRAMEWORK=v4.5
                            set OUTPUT_DIR=.\\
                            set NUNIT_HOME=%NUnit264_32%
                            "%ChocoToolBase%\\OpenCover.Console.exe" -register:administrator -target:"%NUNIT_HOME%\\nunit-console.exe" -targetargs:"%TARGET_TEST_DLL% /nologo /noshadow /framework=%FRAMEWORK%" -filter:"%FILTERS%" -targetdir:"%WSDir%\\Checkout\\bin_test" -output:"%WSDir%\\UT\\results.xml" '''
                        }
                        finally
                        {
                            bat '''REM Create Unit testing and Coverage Report
                            mkdir ..\\Coverage
                            "C:\\ProgramData\\chocolatey\\bin\\ReportGenerator.exe" -reports:"%WSDir%\\UT\\results.xml" -targetdir:"%WSDir%\\Coverage\\coverage_html"'''
                            bat '''move /Y %WSDir%\\UT\\results.xml %WSDir%\\Coverage
                            move /Y %WSDir%\\Checkout\\bin_test\\TestResult.xml %WSDir%\\UT'''
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '$WSDir\\Coverage\\coverage_html', reportFiles: 'index.htm', reportName: 'Coverage Report', reportTitles: ''])
                            nunit testResultsPattern: 'TestResult.xml'
                        }
                    }
                }
            }
        }
    }
}