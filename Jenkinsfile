def skipRemainingStages = false
pipeline {
    agent {
        label 'jenkins-maven'
    }
    stages {
        stage('Check commit msg') {
            when {
                branch 'master'
            }
            steps {
                script {
                    commitMsg = sh (script: 'git log -1 --pretty=%B ${GIT_COMMIT} | head -1', returnStdout: true).trim()
                    echo commitMsg
                    if (commitMsg == 'Bump version') {
                        echo 'jest'
                        skipRemainingStages = true
                    }
                }
            }
        }
        stage('CI Build and push snapshot') {
            when {
                branch 'PR-*'
            }
            steps {
                container('maven') { \
                    sh 'mvn clean test'
                }
            }
        }
        stage('Build Release') {
            when {
                branch 'master'
                expression {
                    !skipRemainingStages
                }
            }
            steps {
                container('maven') {
                    // ensure we're not on a detached head
                    sh 'git checkout master'
                    sh 'git config --global credential.helper store'
                    sh 'jx step git credentials'

                    // so we can retrieve the version in later steps
                    sh "echo \$(jx-release-version) > VERSION"
                    sh "mvn versions:set -DprocessAllModules -DgenerateBackupPoms=false -DnewVersion=\$(cat VERSION)"
                    sh 'mvn clean test'
                    sh "mvn clean deploy"
                    sh 'mvn versions:commit'
                    sh "git commit -m \"Bump version\" ."
                    sh "git push origin master"
                    sh "jx step tag --version \$(cat VERSION)"
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
