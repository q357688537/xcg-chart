node ('chart-slave'){
	def result='';
	def auto =false;
	try{
		stage('SCM') {
			sh 'pwd'			
			checkout scm
			def log = sh (script: "git log -1 | grep '自动化生成chart'", returnStatus: true) 
			if (log == 0) {
				auto=true
				echo "performing build..."
			}
		}
		if(!auto){
			stage('Helm') {		
				container('helm-kubectl') {
					echo "[INFO] Helm clean..."
					sh "rm -rf xcg-*.tgz"
					echo "[INFO] Helm 打包..."
					sh "helm package xcg"
					echo "[INFO] Helm 打包成功."
				}
			}
			
			stage('UpLoad') {
			   withCredentials(bindings: [sshUserPrivateKey(credentialsId: 'github', \
											 keyFileVariable: 'SSH_KEY_FOR_ABC')]) {
					echo "[INFO] 上传"
					sh 'cp ${SSH_KEY_FOR_ABC} ~/.ssh/id_rsa'
					sh 'git config user.email "Jenkins-Robot@example.com"'
					sh 'git config user.name "Jenkins-Robot"'
					sh 'git add -A'
					sh "git commit -m '自动化生成chart'"
					sh 'git push origin HEAD:main'
					echo "[INFO] 上传成功."
				}
			}
		}
		
		result='成功'
	} catch (Exception ex) {
		result='失败'
		//error "build error !!! "
		throw ex
	}finally {
		echo 'I will always say !'+result   
		SendMsg(result) 
	}	
}

def SendMsg(result){
		 def body = [
						msgtype: "text",
						text:[
								content:\
								JOB_NAME+" "+BUILD_DISPLAY_NAME\
								+"\n构建"+result\
								+"\n时间："+BUILD_TIMESTAMP\
								+"\n地址: http://120.79.91.83:8080/job/"+JOB_NAME+"/"+BUILD_NUMBER+"/console\n"\
								],
						at:[isAtAll:false]
				]

				if(result=='失败'){
						body.at.isAtAll=true
				}


				def url='https://oapi.dingtalk.com/robot/send?access_token=387058647f47d80f8c2e3075164b645ed7a35e1ad3735e75ad12ac1a43fed260'
				response = httpRequest consoleLogResponseBody: true\
										, contentType: 'APPLICATION_JSON'\
										, httpMode: 'POST'\
										, requestBody: groovy.json.JsonOutput.toJson(body)\
										, url: url\
										, validResponseCodes: '200'
}