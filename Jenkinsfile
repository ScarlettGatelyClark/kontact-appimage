#!groovy

/*
The MIT License
Copyright (c) 2015-, CloudBees, Inc., and a number of other of contributors
Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:
The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.
        THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
*/

node('linux') {


    currentBuild.result = "SUCCESS"
    properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '3')), disableConcurrentBuilds(), \
    [$class: 'GithubProjectProperty', displayName: '', projectUrlStr: 'https://github.com/sgmoore-appimage-recipes/kontact.git'], pipelineTriggers([[$class: 'GitHubPushTrigger'], pollSCM('H/15 * * * *')])])

    try {

        stage( 'Checkout' ) {
            checkout scm
            checkout changelog: true, poll: false, scm: [$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, \
            extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'appimage-template']], submoduleCfg: [], userRemoteConfigs: [[url: 'https://anongit.kde.org/sysadmin/appimage-tooling']]]
            checkout changelog: true, poll: false, scm: [$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, \
            extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'kmail']], submoduleCfg: [], userRemoteConfigs: [[url: 'https://anongit.kde.org/kmail']]]
       }
        stage( 'Setup' ) {
            sh 'echo 'gem: --no-rdoc --no-ri' >> ~/.gemrc'
            sh 'if [ ! -d "~/.rbenv" ] 
                then
                    echo "Directory ~/.rbenv DOES NOT exists." 
                    git clone https://github.com/sstephenson/rbenv.git ~/.rbenv'
                    git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
                    rbenv install 2.4.1
                    echo 'eval "$(rbenv init -)"' >> ~/.bashrc && rbenv init -
                    cd / && rbenv local 2.4.1 && gem install bundler && ls -l && bundle install --binstubs && bundle show rspec
                fi
            sh 'bundle install'
            def WORKSPACE=pwd()
        }
        stage( 'Build' ) {
            sh 'bundle exec deploy.rb'
       }
       stage('Tests') {
            sh 'find . -type f -exec file {} \\; | grep "not stripped"'
            step([$class: 'LogParserPublisher', failBuildOnError: true, projectRulePath: 'appimage-template/parser.rules', showGraphs: true, unstableOnWarning: true, useProjectRule: true])
      }
   }




    catch (err) {

        currentBuild.result = "FAILURE"

            echo "FAILURE"
        throw err
    }

}
