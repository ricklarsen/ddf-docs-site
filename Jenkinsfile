//"Jenkins Pipeline is a suite of plugins which supports implementing and integrating continuous delivery pipelines into Jenkins. Pipeline provides an extensible set of tools for modeling delivery pipelines "as code" via the Pipeline DSL."
//More information can be found on the Jenkins Documentation page https://jenkins.io/doc/
pipeline {
    agent {
        node {
            label 'linux-small-jdk-js'
            customWorkspace "/jenkins/workspace/${JOB_NAME}/${BUILD_NUMBER}"
        }
    }
    options {
        buildDiscarder(logRotator(numToKeepStr:'25'))
        disableConcurrentBuilds()
        timestamps()
    }
    stages {
        stage('Setup') {
            steps {
                sh 'npm install --global @antora/cli@2.3 @antora/site-generator-default@2.3'
                sh 'npm install --global gitlab:antora/xref-validator'
                sh 'npm install --global antora-site-generator-lunr'
            }
        }
        stage('Test Files') {
            steps {
                sh 'antora --generator @antora/xref-validator site-playbook.yml'
            }
        }
        stage('Build Site') {
            environment {
                DOCSEARCH_ENABLED='true'
                DOCSEARCH_ENGINE='lunr'
            }
            steps {
                sh 'antora site-playbook.yml --generator antora-site-generator-lunr'
            }
        }
        stage('Publish Site') {
            when {
                branch 'add_jenkinsfile'
            }
            steps {
                echo 'Instructions in Jenkinsfile'
                dir('build/site') {
                    stash name: 'site'
                }
                dir('tmp') {
                    // Clone gh-pages branch here
                    // stash name: 'git_folder', includes: './.git'
                    deleteDir()
                    // unstash name: 'git_folder'
                    unstash name: 'site'
                    sh 'ls -a' // Making sure the site is what I would expect
                    //git push 
                }
            }
        }
    }
    post {
        cleanup {
            catchError(buildResult: null, stageResult: 'FAILURE') {
                echo '...Cleaning up workspace'
                cleanWs()
                wrap([$class: 'MesosSingleUseSlave']) {
                    sh 'echo "...Shutting down Jenkins slave: `hostname`"'
                }
            }
        }
    }
}
