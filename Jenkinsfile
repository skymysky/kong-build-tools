pipeline {
    agent none
    environment {
        KONG_SOURCE = "next"
        KONG_SOURCE_LOCATION = "/tmp/kong"
        DOCKERHUB = credentials('dockerhub')
        DOCKER_USERNAME = "${env.DOCKERHUB_USR}"
        DOCKER_PASSWORD = "${env.DOCKERHUB_PSW}"
    }
    stages {
        stage('Build') {
            agent {
                node {
                    label 'docker-compose'
                }
            }
            steps {
                sh 'git clone --single-branch --branch ${KONG_SOURCE} https://github.com/Kong/kong.git ${KONG_SOURCE_LOCATION}'
                sh 'make kong-test-container'
            }
        }
        stage('Test Kong') {
            failFast true
            parallel {
                stage('pdk'){
                    agent {
                        node {
                            label 'docker-compose'
                        }
                    }
                    environment {
                        TEST_SUITE = "pdk"
                    }
                    steps {
                        sh 'git clone --single-branch --branch ${KONG_SOURCE} https://github.com/Kong/kong.git ${KONG_SOURCE_LOCATION}'
                        sh 'make test-kong'
                    }
                }
                stage('postgres'){
                    agent {
                        node {
                            label 'docker-compose'
                        }
                    }
                    environment {
                        TEST_DATABASE = "postgres"
                        TEST_SUITE = "integration"
                    }
                    steps {
                        sh 'git clone --single-branch --branch ${KONG_SOURCE} https://github.com/Kong/kong.git ${KONG_SOURCE_LOCATION}'
                        sh 'make test-kong'
                    }
                }
                stage('postgres plugins'){
                    agent {
                        node {
                            label 'docker-compose'
                        }
                    }
                    environment {
                        TEST_DATABASE = "postgres"
                        TEST_SUITE = "plugins"
                    }
                    steps {
                        sh 'git clone --single-branch --branch ${KONG_SOURCE} https://github.com/Kong/kong.git ${KONG_SOURCE_LOCATION}'
                        sh 'make test-kong'
                    }
                }
                stage('cassandra'){
                    agent {
                        node {
                            label 'docker-compose'
                        }
                    }
                    environment {
                        TEST_DATABASE = "cassandra"
                        TEST_SUITE = "integration"
                    }
                    steps {
                        sh 'git clone --single-branch --branch ${KONG_SOURCE} https://github.com/Kong/kong.git ${KONG_SOURCE_LOCATION}'
                        sh 'make test-kong'
                    }
                }
                stage('cassandra plugins'){
                    agent {
                        node {
                            label 'docker-compose'
                        }
                    }
                    environment {
                        TEST_DATABASE = "cassandra"
                        TEST_SUITE = "plugins"
                    }
                    steps {
                        sh 'git clone --single-branch --branch ${KONG_SOURCE} https://github.com/Kong/kong.git ${KONG_SOURCE_LOCATION}'
                        sh 'make test-kong'
                    }
                }
            }
        }
        stage('Test Release') {
            parallel {
                stage('RedHat Builds'){
                    agent {
                        node {
                            label 'docker-compose'
                        }
                    }
                    environment {
                        PACKAGE_TYPE = "rpm"
                        RESTY_IMAGE_BASE = "rhel"
                        PATH = "/home/ubuntu/bin/:${env.PATH}"
                    }
                    steps {
                        sh 'mkdir -p /home/ubuntu/bin/'
                        sh 'make setup-ci'
                        sh 'git clone --single-branch --branch ${KONG_SOURCE} https://github.com/Kong/kong.git ${KONG_SOURCE_LOCATION}'
                        sh 'export RESTY_IMAGE_TAG=7 && make package-kong'
                        sh 'export RESTY_IMAGE_TAG=8 && make package-kong'
                    }
                    post {
                        success {
                            sh 'export RESTY_IMAGE_TAG=8 && make test'
                        }
                    }
                }
                stage('Centos Builds'){
                    agent {
                        node {
                            label 'docker-compose'
                        }
                    }
                    environment {
                        PACKAGE_TYPE = "rpm"
                        RESTY_IMAGE_BASE = "centos"
                        PATH = "/home/ubuntu/bin/:${env.PATH}"
                    }
                    steps {
                        sh 'mkdir -p /home/ubuntu/bin/'
                        sh 'make setup-ci'
                        sh 'git clone --single-branch --branch ${KONG_SOURCE} https://github.com/Kong/kong.git ${KONG_SOURCE_LOCATION}'
                        sh 'export RESTY_IMAGE_TAG=8 && make package-kong'
                        sh 'export RESTY_IMAGE_TAG=7 && make package-kong'
                        sh 'export RESTY_IMAGE_TAG=6 && make package-kong'
                    }
                    post {
                        success {
                            sh 'export RESTY_IMAGE_TAG=6 && make test'
                        }
                    }
                }
                stage('Debian Builds'){
                    agent {
                        node {
                            label 'docker-compose'
                        }
                    }
                    environment {
                        PACKAGE_TYPE = "deb"
                        RESTY_IMAGE_BASE = "debian"
                        PATH = "/home/ubuntu/bin/:${env.PATH}"
                    }
                    steps {
                        sh 'mkdir -p /home/ubuntu/bin/'
                        sh 'make setup-ci'
                        sh 'git clone --single-branch --branch ${KONG_SOURCE} https://github.com/Kong/kong.git ${KONG_SOURCE_LOCATION}'
                        sh 'export RESTY_IMAGE_TAG=stretch && make package-kong'
                        sh 'export RESTY_IMAGE_TAG=jessie && make package-kong'
                        sh 'export RESTY_IMAGE_TAG=buster && make package-kong'
                        sh 'export RESTY_IMAGE_TAG=bullseye && make package-kong'
                    }
                    post {
                        success {
                            sh 'export RESTY_IMAGE_TAG=bullseye && make test'
                        }
                    }
                }
                stage('Ubuntu Builds'){
                    agent {
                        node {
                            label 'docker-compose'
                        }
                    }
                    environment {
                        PACKAGE_TYPE = "deb"
                        RESTY_IMAGE_BASE = "ubuntu"
                        PATH = "/home/ubuntu/bin/:${env.PATH}"
                        USER = 'travis'
                        AWS_ACCESS_KEY = credentials('AWS_ACCESS_KEY')
                        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
                    }
                    steps {
                        sh 'mkdir -p /home/ubuntu/bin/'
                        sh 'make setup-ci'
                        sh 'git clone --single-branch --branch ${KONG_SOURCE} https://github.com/Kong/kong.git ${KONG_SOURCE_LOCATION}'
                        sh 'export BUILDX=false RESTY_IMAGE_TAG=bionic && make package-kong && make test'
                        sh 'export CACHE=false UPDATE_CACHE=true RESTY_IMAGE_TAG=xenial DOCKER_MACHINE_ARM64_NAME="jenkins-kong-"`cat /proc/sys/kernel/random/uuid` && make setup-build && make package-kong'
                    }
                    post {
                        cleanup {
                            sh 'make cleanup-build'
                        }
                    }
                }
                stage('Other Releases'){
                    agent {
                        node {
                            label 'docker-compose'
                        }
                    }
                    environment {
                        PATH = "/home/ubuntu/bin/:${env.PATH}"
                    }
                    steps {
                        sh 'mkdir -p /home/ubuntu/bin/'
                        sh 'make setup-ci'
                        sh 'git clone --single-branch --branch ${KONG_SOURCE} https://github.com/Kong/kong.git ${KONG_SOURCE_LOCATION}'
                        sh 'PACKAGE_TYPE=src RESTY_IMAGE_BASE=src make package-kong'
                        sh 'PACKAGE_TYPE=apk RESTY_IMAGE_BASE=alpine RESTY_IMAGE_TAG=1 make package-kong'
                        sh 'PACKAGE_TYPE=rpm RESTY_IMAGE_BASE=amazonlinux RESTY_IMAGE_TAG=1 make package-kong'
                    }   
                }
            }
        }
    }
}