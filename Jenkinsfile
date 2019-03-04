#!/usr/bin/env groovy

/**
 * @ Maintainer sudheer veeravalli<veersudhir83@gmail.com>
 * Plugins Needed in Jenkins: Ant, Maven, Ansible, Artifactory, SonarQube,
 * PMD, JUnit, Jacoco, HTML Publisher,
 */

/* Only keep the 10 most recent builds. */
def projectProperties = [
        buildDiscarder(logRotator(artifactDaysToKeepStr: '20', artifactNumToKeepStr: '20', daysToKeepStr: '20', numToKeepStr: '20')),
        [$class: 'GithubProjectProperty', projectUrlStr: 'https://github.com/cloudme2190/devops-web-hackathon.git/']
        ,pipelineTriggers([pollSCM('H/10 * * * *')])
]

properties(projectProperties)

try {

    node {
        if (!isUnix()) {
            sh "echo 'Not a Unix mode'"
        } else {
            def jenkinsIP = '192.168.53.3'
            def mvnHome
            def mvnAnalysisTargets = '-P metrics pmd:pmd test '
            def antHome
            //def ansible
            //def docker
            def artifactoryPublishInfo
            def artifactoryServer
            def isArchivalEnabled = true // params.IS_ARCHIVAL_ENABLED
            // Enable if you want to archive files and configs to artifactory
            def isSonarAnalysisEnabled = true
            //params.IS_ANALYSIS_ENABLED // Enable if you want to analyze code with sonarqube
            def isDeploymentEnabled = true
            //params.IS_DEPLOYMENT_ENABLED // Enable if you want to deploy code on app server
            def isSeleniumTestingEnabled = true
            //params.IS_SELENIUM_TESTING_ENABLED // Enable if you want to generate reports
            def isReportsEnabled = true
            //params.IS_REPORTS_ENABLED // Enable if you want to generate reports

            def appName = 'devops-web-hackathon'// application name currently in progress
            def appEnv  // application environment currently in progress
            def artifactName = appName // name of the war/jar/ear in artifactory
            def artifactExtension = "jar" // extension of the war/jar/ear - for both target directory and artifactory
            def artifactoryRepoName = 'DevOps' // repo name in artifactory
            def artifactoryAppName = appName // application name as per artifactory

            //def tomcatStackName = 'devops-web-hackathon-tomcat'
            def dockerImageName = 'devops-web-hackathon-image'

            def buildNumber = env.BUILD_NUMBER
            def workspaceRoot = env.WORKSPACE
            def artifactoryTempFolder = 'downloadsFromArtifactory'
            // name of the local temp folder where file(s) from artifactory get downloaded
            def sonarHome
            def SONAR_HOST_URL = 'http://localhost:9000'

            // Logic for Slack Notification Service
            def slackBaseUrl = 'https://molina-devops-batch3.slack.com/services/hooks/jenkins-ci/'
            def slackChannel = '#devops_me'
            def slackTeamDomain = 'defaultgrouptalk'
            def slackMessagePrefix = "Hackathon Job ${env.JOB_NAME}:${env.BUILD_NUMBER}"
            def slackTokenCredentialId = '7LHDUUmeNH2JO2NMarLfLxcf'
            // replace with right one from jenkins credentials details

            // color can be good, warning, danger or anything
            //slackSend baseUrl: "${slackBaseUrl}", channel: "${slackChannel}", color: "good", message: "${slackMessagePrefix} -> Build Started", teamDomain: "${slackTeamDomain}", tokenCredentialId: "${slackTokenCredentialId}"


            if (isArchivalEnabled) {
                // Artifactory server id configured in the jenkins along with credentials
                artifactoryServer = Artifactory.server 'Artifactory'
            }

            // functions to use for artifactory operations
            def uploadMavenArtifactUnix = """{
                "files": [
                    {
                        "pattern": "${workspaceRoot}/${appName}/target/${artifactName}.${artifactExtension}",
                        "target": "generic-local/Applications/${artifactoryRepoName}/${artifactoryAppName}/app/${buildNumber}/"
                    }
                ]
            }"""

            stage('Tool Setup') {
                // ** NOTE: These tools must be configured in the jenkins global configuration.
                try {
                    mvnHome = tool name: 'mvn', type: 'maven'
                    antHome = tool name: 'ant', type: 'ant'
                    //ansible = tool name: 'ansible', type: 'org.jenkinsci.plugins.ansible.AnsibleInstallation'
                    //docker = tool name: 'Docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'

                    if (isSonarAnalysisEnabled) {
                        sonarHome = tool name: 'sonar-scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                    }
                    //slackSend color: "good", message: "${slackMessagePrefix} -> Tool Setup Complete"

                } catch (exc) {
                    //slackSend color: "danger", message: "${slackMessagePrefix} -> Tool Setup Failed"
                    error "Failure in Tool Setup stage: ${exc}"
                }
            }

            stage('Checkout') {
                try {
                    //cleanWs() // cleanup workspace before build starts

                    // Checkout codes from repository
                    dir('devops-web-hackathon') {
                        git url: 'https://github.com/cloudme2190/devops-web-hackathon.git/',
                                branch: 'master'
                    }

                    if (isSeleniumTestingEnabled) {
                        dir('devops-hackathon-test-suite') {
                            git url: 'https://github.com/cloudme2190/devops-hackathon-test-suite.git',
                                    branch: 'master'
                        }
                    }

                    if (isReportsEnabled) {
                        dir('devops-static-app') {
                            git url: 'https://github.com/cloudme2190/devops-static-app.git',
                                    branch: 'master'
                        }
                    }

                    dir('downloadsFromArtifactory') {
                        // created folder for artifactory
                    }

                    //slackSend color: "good", message: "${slackMessagePrefix} -> Checkout Complete"
                } catch (exc) {
                    //slackSend color: "danger", message: "${slackMessagePrefix} -> Checkout Failed"
                    error "Failure in Checkout stage: ${exc}"
                }
            }

            stage('Build') {
                try {
                    dir('devops-web-hackathon/') {
                        sh "'${mvnHome}/bin/mvn' clean package"
                        sh "cp ./target/${appName}*.${artifactExtension} ./target/${appName}.${artifactExtension}"
                    }
                    //slackSend color: "good", message: "${slackMessagePrefix} -> Build Complete"
                } catch (exc) {
                    //slackSend color: "danger", message: "${slackMessagePrefix} -> Build Failed"
                    error "Failure in Build stage: ${exc}"
                }
            }

            stage('Analysis') {
                try {
                    if (isSonarAnalysisEnabled) {
                        mvnAnalysisTargets = "${mvnAnalysisTargets} sonar:sonar"
                    }

                    dir('devops-web-hackathon/') {
                        sh "'${mvnHome}/bin/mvn' ${mvnAnalysisTargets}"
                    }

                    //slackSend color: "good", message: "${slackMessagePrefix} -> Analysis Complete"
                } catch (exc) {
                    //slackSend color: "danger", message: "${slackMessagePrefix} -> Analysis Failed"
                    error "Failure in Analysis stage: ${exc}"
                }
            }

            stage('Archive') {
                try {
                    if (isArchivalEnabled) {
                        echo 'Publish Artifacts & appConfig.json in progress'
                        dir('devops-web-hackathon/') {
                            if (fileExists('target/devops-web-hackathon.jar')) {
                                // upload artifactory and also publish build info
                                artifactoryPublishInfo = artifactoryServer.upload(uploadMavenArtifactUnix)
                                artifactoryPublishInfo.retention maxBuilds: 5
                                // and publish build info to artifactory
                                artifactoryServer.publishBuildInfo(artifactoryPublishInfo)
                            } else {
                                error 'Publish: Failed during file upload/publish to artifactory'
                            }
                        }
                    }
                    //slackSend color: "good", message: "${slackMessagePrefix} -> Archival Complete"
                } catch (exc) {
                    //slackSend color: "danger", message: "${slackMessagePrefix} -> Archival Failed"
                    error "Failure in Publish stage: ${exc}"
                }
            }

          
} catch (exc) {
    error "Caught: ${exc}"
}
