def readProperties()
{
        def properties_file_path = "${workspace}" + "@script/properties.yml"
        def property = readYaml file: properties_file_path
        env.APP_NAME = property.APP_NAME
        env.MS_NAME = property.MS_NAME
        env.BRANCH = property.BRANCH
        env.GIT_SOURCE_URL = property.GIT_SOURCE_URL
        env.SONAR_HOST_URL = property.SONAR_HOST_URL
        env.CODE_QUALITY = property.CODE_QUALITY
        env.UNIT_TESTING = property.UNIT_TESTING
        env.CODE_COVERAGE = property.CODE_COVERAGE

}

def FAILED_STAGE
podTemplate(cloud:'openshift',label: 'docker',
  containers: [
    containerTemplate(
      name: 'jnlp',
      image: 'docker:dind',
      alwaysPullImage: true,
     // resourceRequestCpu: '50m',
     // resourceRequestMemory: '500Mi',
      workingDir: '/home/jenkins/agent',

      envVars: [envVar(key:'http_proxy',value:''),envVar(key:'https_proxy',value:'')],
      args: '${computer.jnlpmac} ${computer.name}',
      ttyEnabled: true
    )],volumes: [hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock'),hostPathVolume(hostPath: '/etc/docker/daemon.json', mountPath: '/etc/docker/daemon.json')] )
{
 


node
{
    def MAVEN_HOME = tool "MY_MAVEN"
    def JAVA_HOME = tool "MY_JDK"
    env.PATH= "${env.PATH}${MAVEN_HOME}\\bin;${JAVA_HOME}\\bin"
    
    stage('Checkout')
    {
        readProperties()
        checkout([$class: 'GitSCM', branches: [[name: "*/${BRANCH}"]], doGenerateSubmoduleConfigurations: false, extensions:[], submoduleCfg: [], userRemoteConfigs: [[url: "${GIT_SOURCE_URL}"]]])
    }
    stage('Initial setup')
    {
       
        bat 'mvn clean'
    }
    if (env.UNIT_TESTING == 'True')
    {
        stage('Unit testing')
        {
           
	    
	        
		bat 'mvn test'
	}
	
    }
    if (env.CODE_QUALITY == 'True')
    {
        stage('Code Quality')
        {
            echo 'quality test'
        }
    }
    if (env.CODE_COVERAGE == 'True')
    {
        stage('Coverage testing')
        {
           echo 'mvn cobertura:cobertura'
        }
    }
    if (env.SECURITY_TESTING == 'True')
    {
        stage('Security testing')
        {
            echo 'security'
        }
    }
    if (env.SONAR == 'True')
    {
        stage('sonar testing')
        {
           echo 'mvn sonar:sonar'
        }
    }
post {
	failure {

			echo 'FAIL'
			bat '''curl -g --header "zsessionid":"_7cIVFUMTAe5YRxqNYHuc7obb0aBlXM1WYurWU8" -H "Content-Type":"application/json" -d"{\\"Defect\\":{\\"Name\\":\\"Automated Defect: US2020\\",\\"Severity\\": \\"Cosmetic\\", \\"Priority\\": \\"Resolve Immediately\\", \\"State\\": \\"Open\\"}}" https://rally1.rallydev.com/slm/webservice/v2.0/Defect/create'''
	}
}
    stage('Build and Tag Image for Dev')
   {
//   		script {
//        withCredentials([
//            usernamePassword(credentialsId: 'DockerID', usernameVariable: 'username', passwordVariable: 'password')
//          ])
//        {  
   		
	   FAILED_STAGE=env.STAGE_NAME
	  
	  node('docker')
    {
        
        container('jnlp')
        {
         	checkout([$class: 'GitSCM', branches: [[name: "*/${BRANCH}"]], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: "${SCR_CREDENTIALS}", url: "${GIT_SOURCE_URL}"]]])
            sh "mvn -s Maven/setting clean install -Djacoco.percentage.instruction=0.01"
             /*sh "cp -r /root/.m2/repository/io/prometheus/jmx/jmx_prometheus_javaagent/0.11.0/* ./" */
            sh "docker login ${DOCKER_REGISTRY} -u registryuser -p Inpk2@admregistry"
            sh "docker build -t ${MS_NAME}:latest ."
            sh 'docker tag ${MS_NAME}:latest ${DOCKER_REGISTRY}/${DOCKER_REPO}/${MS_NAME}:redhatdemo-dev-apps'
			sh 'docker push ${DOCKER_REGISTRY}/${DOCKER_REPO}/${MS_NAME}:redhatdemo-dev-apps'
			sh 'docker rmi -f ${DOCKER_REGISTRY}/${DOCKER_REPO}/${MS_NAME}:redhatdemo-dev-apps'
			sh 'docker rmi -f ${MS_NAME}:latest'
        }
    }

}
}

}
