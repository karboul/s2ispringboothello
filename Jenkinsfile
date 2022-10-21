properties([
    parameters([
        choiceParam(name: 'deploymentEnv', choices: ['UAT', 'STG', 'PRD'], description: 'Please Select the Deployment Env'),
    ])   
])

node ('maven') {
    def DEPLOYMENT_SERVER = '13.127.252.94'
    def DEPLOYMENT_USER = 'ubuntu'
    
    def ENVS = ['UAT' : ['isProductionLike' : false, 'deploymentServer' : '13.127.252.94', 'deploymentUser' : 'ubuntu'],
                'STG' : ['isProductionLike' : false, 'deploymentServer' : '', 'deploymentUser' : ''],
                'PRD' : ['isProductionLike' : true, 'deploymentServer' : '', 'deploymentUser' : '']]
    
    stage ('git checkout step') {
        git branch: 'feature/integrate-jfrog', credentialsId: 'github-token', url: 'https://github.com/sachinbhuibar/spring-boot-hello-world.git'
    }
    
    stage ('maven build and deploy step') {
        def TOOL = tool name: 'maven-3.6.3-2', type: 'maven'
        withEnv(["PATH=$TOOL/bin:$PATH"]) {
            withCredentials([usernamePassword(credentialsId: 'jfrog-token', passwordVariable: 'REPO_PASS', usernameVariable: 'REPO_USER')]) {
                sh """
                    mvn --version
                    mvn clean deploy -s settings.xml -Drepo.username=$REPO_USER -Drepo.password=$REPO_PASS
                """
            }
        }
    }
    
    stage ('deploy the application') {
        try {
            sh """
                ssh ${ENVS[params.deploymentEnv]['deploymentUser']}@${ENVS[params.deploymentEnv]['deploymentServer']} sudo systemctl stop spring-boot-hello-world.service
            """
        }
        catch(e) {
            
        }
        finally {
            sh """
                echo 'put target/spring-boot-hello-world-0.0.1-SNAPSHOT.jar deployable/' | sftp ${ENVS[params.deploymentEnv]['deploymentUser']}@${ENVS[params.deploymentEnv]['deploymentServer']}
                ssh -q ${ENVS[params.deploymentEnv]['deploymentUser']}@${ENVS[params.deploymentEnv]['deploymentServer']} sudo systemctl start spring-boot-hello-world.service
            """
        }
    }
    
}
