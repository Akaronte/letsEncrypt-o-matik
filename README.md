# letsEncrypt-o-matik
Create signed certificate only with a public dns and docker

Firts you need a docker with a 80 and 8080 port avaible.

Use the next jenkinsfile with two parameters:
 - mail
 - domain
 
This job create de ssl certificates and export in jenkins like a artifact.

When you mount a jenkins use: 

docker run --name jenkins-docker -v /var/run/docker.sock:/var/run/docker.sock -p 8080:8080 -d akaronte/jenkins-docker

if you want a persistence data mount a volume for jenkins_home

docker run --name jenkins-docker -v /var/run/docker.sock:/var/run/docker.sock -p 8080:8080 -v /jenkins_home:/var/jenkins_home -d akaronte/jenkins-docker

```
pipeline {
    agent any
        parameters {
            string(name: 'domain', defaultValue: '', description: 'hostname')
            string(name: 'mail', defaultValue: '', description: 'mail')
        }
    stages {
        stage('Docker pull') {
            steps {
            sh "docker pull akaronte/certbot"
            }
        }
        stage('Run certbot') {
            agent {
                docker {
                    image 'akaronte/certbot'
                    args "-u root -p 80:80 -v ${WORKSPACE}:/mnt"
                }
            }
            steps {
                sh """
                /usr/sbin/nginx
                certbot certonly --email ${params.mail} --agree-tos --non-interactive --webroot -w "/var/www/html" -d ${params.domain}
                cp -r /etc/letsencrypt/archive/${params.domain}/* ${WORKSPACE}
                cd ${WORKSPACE}
                ls
                chmod -R 666 *
                cp fullchain1.pem ${params.domain}.crt
                cp fullchain1.pem ${params.domain}.crt.remove
                cp privkey1.pem	${params.domain}.key
                
                """
                archiveArtifacts artifacts: '*.pem'
                archiveArtifacts artifacts: '*.crt'
                archiveArtifacts artifacts: '*.key'
            }
        }
    }
    post{
        cleanup {
            echo "CLEAN"
            cleanWs()
        }
    }
}
```
