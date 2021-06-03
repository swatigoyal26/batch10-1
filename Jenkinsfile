try{
    node{
        def mavenHome
        def mavenCMD
        def docker
        def dockerCMD
        def tagName = "1.0"
        
        stage('Preparation'){
            echo "Preparing the Jenkins environment with required tools..."
            mavenHome = tool name: 'maven 3', type: 'maven'
            mavenCMD = "${mavenHome}/bin/mvn"
            docker = tool name: 'docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
            dockerCMD = "$docker/bin/docker"
        }
        
        stage('git checkout'){
            echo "Checking out the code from git repository..."
            git 'https://github.com/swatigoyal26/batch10-1.git'
        }
        
        stage('Build, Test and Package'){
            echo "Building the addressbook application..."
            sh "${mavenCMD} clean package"
        }
        
        stage('Sonar check'){
            echo "scanning the app"
            tool name:'maven 3',type: 'maven'
            sh 'mvn sonar:sonar -Dsonar.projectKey=sonar -Dsonar.login=admin -Dsonar.password=admin123 -Dsonar.host.url=http://ec2-35-172-232-144.compute-1.amazonaws.com'
        }  
        
        stage('Integration test'){
            echo "Executing Regression Test Suits..."
            // command to execute selenium test suits
        }
        
        stage('publish report'){
            echo " Publishing HTML report.."
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: ''])
        }
        
        stage('Build Docker Image'){
            echo "Building docker image for javabootcamp application ..."
            sh "${dockerCMD} build -t puneet20/javabootcamp:${tagName} ."
        }
        
        stage("Push Docker Image to Docker Registry"){
            echo "Pushing image to docker hub"
            withCredentials([string(credentialsId: 'dockerPwd123', variable: 'dockerHubPwd')]) {
              sh "${dockerCMD} login -u puneet20 -p ${dockerHubPwd}"
              sh "${dockerCMD} push puneet20/javabootcamp:${tagName}"
            }
        }
        
        stage('Deploy Application'){
            echo "Installing desired software.."
            echo "Bring docker service up and running"
            echo "Deploying addressbook application"
            ansiblePlaybook credentialsId: 'ansible-ssh', disableHostKeyChecking: true, installation: 'ansible 2.9.22', inventory: '/etc/ansible/hosts', playbook: 'deploy-playbook.yml'
        }
        stage('Clean up'){
            echo "Cleaning up the workspace..."
            emailext body: 'Your build has been successfully run', subject: 'Build Result', to: 'puneetagrawal.40@gmail.com'
            cleanWs()
        }
    }
}
catch(Exception err){
    echo "Exception occured..."
    echo err
    currentBuild.result="FAILURE"
    emailext body: 'Your build has been failed unsuccessful', subject: 'Build Result', to: 'puneetagrawal.40@gmail.com'
}
finally {
    (currentBuild.result!= "ABORTED") && node("master") {
        echo "finally gets executed and end an email notification for every build"
        emailext body: 'Your build has been successful or unsuccessful', subject: 'Build Result', to: 'puneetagrawal.40@gmail.com'
    }
    
}
