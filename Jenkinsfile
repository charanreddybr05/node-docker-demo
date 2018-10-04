pipeline {
    stages {
        stage('Clone repository') {
            agent any
            steps {
                checkout scm
            }
			steps {
			sh 'docker build -t nodejs:v1 .'
			}
        }
        stage('NodeJs Build') {
            agent {
            steps {
                parallel(
					a: {
					  sh 'docker run --rm -p 3000:3000 -d -v $(pwd)/app:/src/app -v $(pwd)/public:/src/public --link nd-db --name nd-app1 node-docker'
					},
					b: {
					  sh 'docker run --rm -p 3001:3000 -d -v $(pwd)/app:/src/app -v $(pwd)/public:/src/public --link nd-db --name nd-app2 node-docker'
					}
			)
            }
        }
		}
        stage('Creating Nginx') {
            agent any
            steps {
                parallel(
					a: {
					  sh 'docker run --name docker-nginx1 -p 81:80 -d -v ~/nginx.conf:/etc/nginx/conf/nginx.conf nginx'
					},
					b: {
					  sh 'docker run --name docker-nginx1 -p 82:80 -d -v ~/nginx.conf:/etc/nginx/conf/nginx.conf nginx'
					}
			)
            }
        }
		Stage('Validation') {
			agent any
			steps {
				sh 'curl -I 172.22.177.51:80 > node1.json'
				def OUTPUT = cat node1.json | grep 200
				//if ( -z "${OUTPUT}"){
				//	echo "Couldn't access tha application"
				//	sh 'exit 1'
				//}
				
			}
			steps {
				sh 'curl -I 172.22.177.51:81 > node2.json'
				def OUTPUT = sh 'cat node2.json | grep 200'
				//if ("${OUTPUT}" == "200" ) {
				//	echo "Couldn't access tha application"
				//	sh 'exit 1'
				//}
				
			}
		}

        stage('Destroy containers') {
            agent any
        steps {
			sh 'docker kill $(docker ps -aq) '
        }
  }
}
}
