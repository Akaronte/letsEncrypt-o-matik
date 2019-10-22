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
