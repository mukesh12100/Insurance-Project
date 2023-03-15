node{
    def mavenHome
    def mavenCMD
    def docker
    def dockerCMD
    def tagName
    
    stage('environment creation')
        echo 'initialize all the variables'
        mavenHome = tool name: 'maven' , type:'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
        docker = tool name: 'docker' , type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
        dockerCMD = "${docker}/bin/docker"
        tagName="5.0"
        
        
    stage ('code checking out'){
        try{
            echo 'checkout the code from git repository'
            git 'https://github.com/mukesh12100/Insurance-Project.git'
        }
        catch(Exception e){
            echo 'Exception occurred in Git Code Checkout Stage'
            currentBuild.result = "FAILURE"
            emailext body: '''Dear All,
            The Jenkins job ${JOB_NAME} has been failed. Request you to please look at it immediately by clicking on the below link.
            ${BUILD_URL}''', subject: 'Job ${JOB_NAME} ${BUILD_NUMBER} is failed', to: 'mukeshpawar12100@gmail.com'
        }
  }
    stage('build the Application'){
        echo "Cleaning... Compiling... Testing... Packing..."
        sh 'mvn clean package'
        sh "${mavenCMD} clean package"
    }
    
    stage('report generation'){
        echo "Generating the testing report"
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '/var/lib/jenkins/workspace/Insure-Me/target/surefire-reports', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        
    }
    
    stage('making image/ containerise'){
        echo "containerising the application"
        sh "${dockerCMD} build -t mukesh12100/insure-me:${tagName} ."
    }
    
    stage ('pushing image to docker registry'){
        echo 'pushing the image'
        withCredentials([string(credentialsId: 'dockerpass', variable: 'dockerhubpassword')]) {
        sh "${dockerCMD} login -u mukesh12100 -p ${dockerhubpassword}"
        sh "${dockerCMD} push mukesh12100/insure-me:${tagName}"

    }
    
    stage('configure and deploy to the test server'){
        ansiblePlaybook become: true, credentialsId: 'ansible-key', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/hosts', playbook: 'ansible-playbook.yml', vaultCredentialsId: 'dockerpass'
        
    }
    }
    
    }
