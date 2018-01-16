#!/usr/bin/env groovy
ghc_apk = [:]

// Until I debug the arm issue, don't build armhf by default
if (['8.0.2'].contains(scm.branches[0].name)) {
  default_arm = true
} else {
  default_arm = false
}

pipeline {
  agent { label "alpine" }
  parameters {
    choice(choices: 'all\nghc\ncabal\nstack', name: 'action'
           , description: 'TODO: I do nothing right now. What to build, default is to try building everything')
    string(defaultValue: 'https://github.com/mitchty/aports.git', name: 'aports_url'
           , description: 'Url for the aports repo')
    booleanParam(defaultValue: false, name: 'git'
                 , description: 'TODO: I do nothing right now. Build ghc off of the upstream git repo?')
    booleanParam(defaultValue: false, name: 'apk_update'
                 , description: 'Update apk packages/db prior to building stuff?')
    booleanParam(defaultValue: true, name: 'x86_64'
                 , description: 'Build x86_64?')
    booleanParam(defaultValue: default_arm, name: 'armhf'
                 , description: 'Build armhf')
  }
  stages {
    stage("pre-setup") {
      steps {
        script {
          if (env.BRANCH_NAME == 'master') { exit 0 }
          if (armhf != "true" && x86_64 != "true") { error("Have to pick at least one of the arches to build") }
          if (armhf == "true") {
            ghc_apk['armhf ghc'] = { script { timeout(time: 2, unit: 'DAYS') {
              node('alpine-armhf') {
                lock("armhf") {
                  deleteDir()
  		if (apk_update == "true") {
  	          sh "/sbin/apk update && /sbin/apk upgrade"
  		}
                  git branch: env.BRANCH_NAME, credentialsId: 'mitchty_github', url: aports_url
                  sh "chown -R build:build ${env.WORKSPACE}"
                  sh "su -l build -c 'cd ${env.WORKSPACE}/community/ghc && abuild checksum'"
                  sh "su -l build -c 'ulimit -c unlimited; cd ${env.WORKSPACE}/community/ghc && abuild -r'"
                  dir("${env.WORKSPACE}/community/ghc") {
                    archiveArtifacts artifacts: "ghc*.tar.xz", fingerprint: true
                    sh "mv APKBUILD APKBUILD.armhf"
                    archiveArtifacts artifacts: "APKBUILD.armhf", fingerprint: true
                  }
                }
              }
            }}}
          }
          if (x86_64 == "true") {
            ghc_apk['x86_64 ghc'] = { script { timeout(time: 2, unit: 'HOURS') {
              node('alpine-x86_64') {
                lock("x86_64") {
                  deleteDir()
  		if (apk_update == "true") {
  	          sh "/sbin/apk update && /sbin/apk upgrade"
  		}
                  git branch: env.BRANCH_NAME, credentialsId: 'mitchty_github', url: aports_url
                  sh "chown -R build:build ${env.WORKSPACE}"
                  sh "su -l build -c 'cd ${env.WORKSPACE}/community/ghc && abuild checksum'"
                  sh "su -l build -c 'cd ${env.WORKSPACE}/community/ghc && abuild -r'"
                  dir("${env.WORKSPACE}/community/ghc") {
                    sh "mv APKBUILD APKBUILD.x86_64"
                    archiveArtifacts artifacts: "APKBUILD.x86_64", fingerprint: true
                  }
                  sh "su -l build -c 'cd ${env.WORKSPACE}/community/ghc && abuild bindist'"
                  dir("${env.WORKSPACE}/community/ghc") {
                    archiveArtifacts artifacts: "ghc*.tar.xz", fingerprint: true
                  }
                }
              }
            }}}
          }
        }
      }
    }
    stage("ghc apk") {
      steps {
        script {
          parallel(ghc_apk)
        }
      }
    }
  }
  post {
    success {
      deleteDir()
    }
  }
  options {
    buildDiscarder(logRotator(numToKeepStr: '31', artifactNumToKeepStr: '3'))
    timestamps()
  }
}
