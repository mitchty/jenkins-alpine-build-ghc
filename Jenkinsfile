#!/usr/bin/env groovy

pipeline {
  agent { label "alpine" }
  parameters {
    choice(choices: 'all\nghc\ncabal\nstack', name: 'action'
           , description: 'What to build, default is to try building everything')
    string(defaultValue: 'master', name: 'aports_branch'
           , description: 'What branch to build for aports')
    string(defaultValue: 'https://github.com/mitchty/aports', name: 'aports_url'
           , description: 'Url for the aports repo')
    booleanParam(defaultValue: false, name: 'with_bindist'
                 , description: 'Build ghc bindists? Default is not to')
    booleanParam(defaultValue: false, name: 'git'
                 , description: 'Build ghc off of the upstream git repo?')
  }
  stages {
    stage("foo") {
      steps {
        node('alpine-armhf') {
          sh 'uname -a'
        }
        node('alpine-x86_64') {
          sh 'uname -a'
        }
      }
    }
  }
}
