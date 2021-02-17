def notify(message) {
    def token = "r7LCSGxlKGQmYDPBnEZhS3GLYbg6QRP71DWd8VQ1ge7";
    def jobName = env.JOB_NAME + ' - ' + env.BRANCH_NAME;
    def buildNo = env.BUILD_NUMBER;
      
    def url = "https://notify-api.line.me/api/notify";
    def lineMessage = "${jobName} [#${buildNo}] : ${message} \r\n";
    sh "curl ${url} -H 'Authorization: Bearer ${token}' -F 'message=${lineMessage}'";
}

pipeline {
    agent any
    stages {
        stage ("Change Directory") {
            steps {
                script {
                    try{
                        sh 'cd /not-found-directory';
                    }catch(Exception e){
                        notify("${e.getMessage()}");
                        throw e;
                    }
                }
                
            }    
        }
        
    }
}
