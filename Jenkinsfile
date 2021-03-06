#!groovy

pipeline {
    agent any
    options {
        timestamps()
        timeout(time: 2, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '30', artifactNumToKeepStr: '5'))
        disableConcurrentBuilds()
    }
    environment {
        // In case another branch beside master or develop should be deployed, enter it here
        BRANCH_TO_DEPLOY = 'windows-upload'
        GITHUB_TOKEN = credentials('cdc81429-53c7-4521-81e9-83a7992bca76')
        SPECTRECOIN_RELEASE = "2.2.0"
        DISCORD_WEBHOOK = credentials('991ce248-5da9-4068-9aea-8a6c2c388a19')
    }
    stages {
        stage('Notification') {
            steps {
                // Using result state 'ABORTED' to mark the message on discord with a white border.
                // Makes it easier to distinguish job-start from job-finished
                discordSend(
                        description: "Started build #$env.BUILD_NUMBER",
                        image: '',
                        link: "$env.BUILD_URL",
                        successful: true,
                        result: "ABORTED",
                        thumbnail: 'https://wiki.jenkins-ci.org/download/attachments/2916393/headshot.png',
                        title: "$env.JOB_NAME",
                        webhookURL: "${DISCORD_WEBHOOK}"
                )
            }
        }
        stage('Latest binary archives') {
            when {
                anyOf { branch 'develop'; branch "${BRANCH_TO_DEPLOY}" }
            }
            parallel {
                stage('Debian') {
                    agent {
                        label "docker"
                    }
                    steps {
                        script {
                            // Copy step on Dockerfile is not working if Dockerfile is not located on root dir!
                            // So copy required Dockerfile to root dir for each build
                            sh "cp ./Debian/binary/Dockerfile ."
                            docker.build(
                                    "spectreproject/spectre-distribution-debian",
                                    "--rm --build-arg GITHUB_TOKEN=${GITHUB_TOKEN} --build-arg SPECTRECOIN_REPOSITORY=spectre --build-arg REPLACE_EXISTING_ARCHIVE=--replace ."
                            )
                            sh "rm Dockerfile"
                        }
                    }
                    post {
                        always {
                            sh "docker system prune --all --force"
                        }
                    }
                }
/*
                stage('CentOS') {
                    agent {
                        label "docker"
                    }
                    steps {
                        script {
                            // Copy step on Dockerfile is not working if Dockerfile is not located on root dir!
                            // So copy required Dockerfile to root dir for each build
                            sh "cp ./CentOS/binary/Dockerfile ."
                            docker.build(
                                    "spectreproject/spectre-distribution-centos",
                                    "--rm --build-arg GITHUB_TOKEN=${GITHUB_TOKEN} --build-arg SPECTRECOIN_REPOSITORY=spectre --build-arg REPLACE_EXISTING_ARCHIVE=--replace ."
                            )
                            sh "rm Dockerfile"
                        }
                    }
                    post {
                        always {
                            sh "docker system prune --all --force"
                        }
                    }
                }
*/
                stage('Fedora') {
                    agent {
                        label "docker"
                    }
                    steps {
                        script {
                            // Copy step on Dockerfile is not working if Dockerfile is not located on root dir!
                            // So copy required Dockerfile to root dir for each build
                            sh "cp ./Fedora/binary/Dockerfile ."
                            docker.build(
                                    "spectreproject/spectre-distribution-fedora",
                                    "--rm --build-arg GITHUB_TOKEN=${GITHUB_TOKEN} --build-arg SPECTRECOIN_REPOSITORY=spectre --build-arg REPLACE_EXISTING_ARCHIVE=--replace ."
                            )
                            sh "rm Dockerfile"
                        }
                    }
                    post {
                        always {
                            sh "docker system prune --all --force"
                        }
                    }
                }
                stage('Raspberry Pi') {
                    agent {
                        label "docker"
                    }
                    steps {
                        script {
                            // Copy step on Dockerfile is not working if Dockerfile is not located on root dir!
                            // So copy required Dockerfile to root dir for each build
                            sh "cp ./RaspberryPi/binary/Dockerfile ."
                            docker.build(
                                    "spectreproject/spectre-distribution-raspi",
                                    "--rm --build-arg GITHUB_TOKEN=${GITHUB_TOKEN} --build-arg SPECTRECOIN_REPOSITORY=spectre --build-arg REPLACE_EXISTING_ARCHIVE=--replace ."
                            )
                            sh "rm Dockerfile"
                        }
                    }
                    post {
                        always {
                            sh "docker system prune --all --force"
                        }
                    }
                }
                stage('Ubuntu') {
                    agent {
                        label "docker"
                    }
                    steps {
                        script {
                            // Copy step on Dockerfile is not working if Dockerfile is not located on root dir!
                            // So copy required Dockerfile to root dir for each build
                            sh "cp ./Ubuntu/binary/Dockerfile ."
                            docker.build(
                                    "spectreproject/spectre-distribution-ubuntu",
                                    "--rm --build-arg GITHUB_TOKEN=${GITHUB_TOKEN} --build-arg SPECTRECOIN_REPOSITORY=spectre --build-arg REPLACE_EXISTING_ARCHIVE=--replace ."
                            )
                            sh "rm Dockerfile"
                        }
                    }
                    post {
                        always {
                            sh "docker system prune --all --force"
                        }
                    }
                }
                stage('Windows') {
                    agent {
                        label "docker"
                    }
                    steps {
                        script {
                            sh "wget https://ci.spectreproject.io/job/Spectrecoin/job/spectre/job/develop/lastSuccessfulBuild/artifact/Spectrecoin-latest.zip"
                            sh "docker run \\\n" +
                                    "--rm \\\n" +
                                    "-e GITHUB_TOKEN=${GITHUB_TOKEN} \\\n" +
                                    "-v ${WORKSPACE}:/filesToUpload \\\n" +
                                    "spectreproject/github-uploader:latest \\\n" +
                                    "github-release upload \\\n" +
                                    "    --user spectrecoin \\\n" +
                                    "    --repo spectre \\\n" +
                                    "    --tag latest \\\n" +
                                    "    --name \"Spectrecoin-latest-WIN64.zip\" \\\n" +
                                    "    --file /filesToUpload/Spectrecoin-latest.zip \\\n" +
                                    "    --replace"
                        }
                    }
                    post {
                        always {
                            sh "docker system prune --all --force"
                        }
                    }
                }
            }
        }
        stage('Release binary archives') {
            when {
                branch 'master'
            }
            parallel {
                stage('Debian') {
                    agent {
                        label "docker"
                    }
                    steps {
                        script {
                            // Copy step on Dockerfile is not working if Dockerfile is not located on root dir!
                            // So copy required Dockerfile to root dir for each build
                            // Additionally replace tag 'latest' with release version to use proper docker
                            // image to grab binaries from
                            sh "sed 's#latest /usr/local/bin/spectrecoin#${SPECTRECOIN_RELEASE} /usr/local/bin/spectrecoin#g' ./Debian/binary/Dockerfile > ./Dockerfile"
                            docker.build(
                                    "spectreproject/spectre-distribution-debian",
                                    "--rm --build-arg GITHUB_TOKEN=${GITHUB_TOKEN} --build-arg SPECTRECOIN_RELEASE=${SPECTRECOIN_RELEASE} --build-arg REPLACE_EXISTING_ARCHIVE=--replace ."
                            )
                            sh "rm Dockerfile"
                        }
                    }
                    post {
                        always {
                            sh "docker system prune --all --force"
                        }
                    }
                }
/*
                stage('CentOS') {
                    agent {
                        label "docker"
                    }
                    steps {
                        script {
                            // Copy step on Dockerfile is not working if Dockerfile is not located on root dir!
                            // So copy required Dockerfile to root dir for each build
                            // Additionally replace tag 'latest' with release version to use proper docker
                            // image to grab binaries from
                            sh "sed 's#latest /usr/local/bin/spectrecoin#${SPECTRECOIN_RELEASE} /usr/local/bin/spectrecoin#g' ./CentOS/binary/Dockerfile > ./Dockerfile"
                            docker.build(
                                    "spectreproject/spectre-distribution-centos",
                                    "--rm --build-arg GITHUB_TOKEN=${GITHUB_TOKEN} --build-arg SPECTRECOIN_RELEASE=${SPECTRECOIN_RELEASE} --build-arg REPLACE_EXISTING_ARCHIVE=--replace ."
                            )
                            sh "rm Dockerfile"
                        }
                    }
                    post {
                        always {
                            sh "docker system prune --all --force"
                        }
                    }
                }
*/
                stage('Fedora') {
                    agent {
                        label "docker"
                    }
                    steps {
                        script {
                            // Copy step on Dockerfile is not working if Dockerfile is not located on root dir!
                            // So copy required Dockerfile to root dir for each build
                            // Additionally replace tag 'latest' with release version to use proper docker
                            // image to grab binaries from
                            sh "sed 's#latest /usr/local/bin/spectrecoin#${SPECTRECOIN_RELEASE} /usr/local/bin/spectrecoin#g' ./Fedora/binary/Dockerfile > ./Dockerfile"
                            docker.build(
                                    "spectreproject/spectre-distribution-fedora",
                                    "--rm --build-arg GITHUB_TOKEN=${GITHUB_TOKEN} --build-arg SPECTRECOIN_RELEASE=${SPECTRECOIN_RELEASE} --build-arg REPLACE_EXISTING_ARCHIVE=--replace ."
                            )
                            sh "rm Dockerfile"
                        }
                    }
                    post {
                        always {
                            sh "docker system prune --all --force"
                        }
                    }
                }
                stage('Raspberry Pi') {
                    agent {
                        label "docker"
                    }
                    steps {
                        script {
                            // Copy step on Dockerfile is not working if Dockerfile is not located on root dir!
                            // So copy required Dockerfile to root dir for each build
                            // Additionally replace tag 'latest' with release version to use proper docker
                            // image to grab binaries from
                            sh "sed 's#latest /usr/local/bin/spectrecoin#${SPECTRECOIN_RELEASE} /usr/local/bin/spectrecoin#g' ./RaspberryPi/binary/Dockerfile > ./Dockerfile"
                            docker.build(
                                    "spectreproject/spectre-distribution-raspi",
                                    "--rm --build-arg GITHUB_TOKEN=${GITHUB_TOKEN} --build-arg SPECTRECOIN_RELEASE=${SPECTRECOIN_RELEASE} --build-arg REPLACE_EXISTING_ARCHIVE=--replace ."
                            )
                            sh "rm Dockerfile"
                        }
                    }
                    post {
                        always {
                            sh "docker system prune --all --force"
                        }
                    }
                }
                stage('Ubuntu') {
                    agent {
                        label "docker"
                    }
                    steps {
                        script {
                            // Copy step on Dockerfile is not working if Dockerfile is not located on root dir!
                            // So copy required Dockerfile to root dir for each build
                            // Additionally replace tag 'latest' with release version to use proper docker
                            // image to grab binaries from
                            sh "sed 's#latest /usr/local/bin/spectrecoin#${SPECTRECOIN_RELEASE} /usr/local/bin/spectrecoin#g' ./Ubuntu/binary/Dockerfile > ./Dockerfile"
                            docker.build(
                                    "spectreproject/spectre-distribution-ubuntu",
                                    "--rm --build-arg GITHUB_TOKEN=${GITHUB_TOKEN} --build-arg SPECTRECOIN_RELEASE=${SPECTRECOIN_RELEASE} --build-arg REPLACE_EXISTING_ARCHIVE=--replace ."
                            )
                            sh "rm Dockerfile"
                        }
                    }
                    post {
                        always {
                            sh "docker system prune --all --force"
                        }
                    }
                }
                stage('Windows') {
                    agent {
                        label "docker"
                    }
                    steps {
                        script {
                            sh "wget https://ci.spectreproject.io/job/Spectrecoin/job/spectre/job/master/lastSuccessfulBuild/artifact/Spectrecoin-${SPECTRECOIN_RELEASE}.zip"
                            sh "docker run \\\n" +
                                    "--rm \\\n" +
                                    "-e GITHUB_TOKEN=${GITHUB_TOKEN} \\\n" +
                                    "-v ${WORKSPACE}:/filesToUpload \\\n" +
                                    "spectreproject/github-uploader:latest \\\n" +
                                    "github-release upload \\\n" +
                                    "    --user spectrecoin \\\n" +
                                    "    --repo spectre \\\n" +
                                    "    --tag latest \\\n" +
                                    "    --name \"Spectrecoin-${SPECTRECOIN_RELEASE}-WIN64.zip\" \\\n" +
                                    "    --file /filesToUpload/Spectrecoin-${SPECTRECOIN_RELEASE}.zip \\\n" +
                                    "    --replace"
                        }
                    }
                    post {
                        always {
                            sh "docker system prune --all --force"
                        }
                    }
                }
            }
        }
    }
    post {
        success {
            script {
                if (!hudson.model.Result.SUCCESS.equals(currentBuild.getPreviousBuild()?.getResult())) {
                    emailext(
                            subject: "GREEN: '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                            body: '${JELLY_SCRIPT,template="html"}',
                            recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
//                            to: "to@be.defined",
//                            replyTo: "to@be.defined"
                    )
                }
                discordSend(
                        description: "Build #$env.BUILD_NUMBER finished successfully",
                        image: '',
                        link: "$env.BUILD_URL",
                        successful: true,
                        thumbnail: 'https://wiki.jenkins-ci.org/download/attachments/2916393/headshot.png',
                        title: "$env.JOB_NAME",
                        webhookURL: "${DISCORD_WEBHOOK}"
                )
            }
        }
        unstable {
            emailext(
                    subject: "YELLOW: '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                    body: '${JELLY_SCRIPT,template="html"}',
                    recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
//                    to: "to@be.defined",
//                    replyTo: "to@be.defined"
            )
            discordSend(
                    description: "Build #$env.BUILD_NUMBER finished unstable",
                    image: '',
                    link: "$env.BUILD_URL",
                    successful: true,
                    result: "UNSTABLE",
                    thumbnail: 'https://wiki.jenkins-ci.org/download/attachments/2916393/headshot.png',
                    title: "$env.JOB_NAME",
                    webhookURL: "${DISCORD_WEBHOOK}"
            )
        }
        failure {
            emailext(
                    subject: "RED: '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                    body: '${JELLY_SCRIPT,template="html"}',
                    recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
//                    to: "to@be.defined",
//                    replyTo: "to@be.defined"
            )
            discordSend(
                    description: "Build #$env.BUILD_NUMBER failed!",
                    image: '',
                    link: "$env.BUILD_URL",
                    successful: false,
                    thumbnail: 'https://wiki.jenkins-ci.org/download/attachments/2916393/headshot.png',
                    title: "$env.JOB_NAME",
                    webhookURL: "${DISCORD_WEBHOOK}"
            )
        }
    }
}