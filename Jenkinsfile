pipeline {

    options { disableConcurrentBuilds() }

    environment {
        // TAG_NAME = "${env.BRANCH_NAME == 'master' ? 'stable' : 'test'}"
        TAG_NAME = 'latest'
        REGISTRY_CREDENTIALS_ID = 'jenkins-registry-admin'
    }

    agent { label "master" }

    stages {

        stage('RAW CONFIG HANDLING') {
            steps {
                sh "ls -lrt && pwd"
                sh "mkdir -p ${JENKINS_HOME}/jenkins_config/"
                sh "cp -rf * ${JENKINS_HOME}/jenkins_config/"
                

            }
        }

        stage('CONSUL TEMPLATE') {
            steps { sh '${JENKINS_HOME}/consul-template -vault-addr "http://consul.es-jr.cn" -config "jenkins_config.hcl" -once -vault-retry-attempts=1 -vault-renew-token=false' }
        }

        stage('RUN GROOVY on master') {
            steps {
                load("/var/jenkins_home/jenkins_config/src/kubernetes.groovy")
            }
        }

        stage('Build dockerfiles') {
            steps {
                script {
                    docker.withRegistry('registry.apps.es1688.k8s.local', "${REGISTRY_CREDENTIALS_ID}") {
                        docker.build("georgesre/jnlp-node-team1:${TAG_NAME}", "-f dockerfiles/node-team1/Dockerfile dockerfiles/node-team1").push()
                    }
                    docker.withRegistry('registry.apps.es1688.k8s.local', "${REGISTRY_CREDENTIALS_ID}") {
                        docker.build("georgesre/jnlp-node-team2:${TAG_NAME}", "-f dockerfiles/node-team2/Dockerfile dockerfiles/node-team2").push()
                    }
                    docker.withRegistry('registry.apps.es1688.k8s.local', "${REGISTRY_CREDENTIALS_ID}") {
                        docker.build("georgesre/jnlp-node-team3:${TAG_NAME}", "-f dockerfiles/node-team3/Dockerfile dockerfiles/node-team3").push()
                    }
                    docker.withRegistry('registry.apps.es1688.k8s.local', "${REGISTRY_CREDENTIALS_ID}") {
                        docker.build("georgesre/jnlp-node-team4:${TAG_NAME}", "-f dockerfiles/node-team4/Dockerfile dockerfiles/node-team4").push()
                    }
                }
            }
        }

        stage('Run Parallel Tests') {
            parallel {
                stage('Test On node-team1') {
                    agent { label "node-team1" }
                    steps { sh "echo I am good on node-team1" }
                }
                stage('Test On node-team2') {
                    agent { label "node-team2" }
                    steps { sh "echo I am good on node-team2" }
                }
                stage('Test On node-team3') {
                    agent { label "node-team3" }
                    steps { sh "echo I am good on node-team3" }
                }
                stage('Test On node-team4') {
                    agent { label "node-team4" }
                    steps { sh "echo I am good on node-team4 && perl -v" }
                }
            }
        }
    }
}
