/*
    zproject - Project

    Copyright (c) the Contributors as noted in the AUTHORS file.
    This file is part of CZMQ, the high-level C binding for 0MQ:
    http://czmq.zeromq.org.

    This Source Code Form is subject to the terms of the Mozilla Public
    License, v. 2.0. If a copy of the MPL was not distributed with this
    file, You can obtain one at http://mozilla.org/MPL/2.0/.
*/

pipeline {
    agent any
    triggers {
        pollSCM 'H/5 * * * *'
    }
    stages {
        stage ('prepare') {
            steps {
                sh './autogen.sh'
                stash (name: 'prepped')
            }
        }
        stage ('compile') {
            parallel {
                stage ('build with DRAFT') {
                    steps {
                        unstash (name: 'prepped')
                        sh './configure --enable-drafts=yes'
                        sh 'make -k -j4 || make'
                        sh 'echo "Are GitIgnores good after make with drafts? (should have no output below)"; git status -s || true'
                        stash (name: 'built-draft')
                    }
                }
                stage ('build without DRAFT') {
                    steps {
                        unstash (name: 'prepped')
                        sh './configure --enable-drafts=no'
                        sh 'make -k -j4 || make'
                        sh 'echo "Are GitIgnores good after make without drafts? (should have no output below)"; git status -s || true'
                        stash (name: 'built-nondraft')
                    }
                }
                stage ('build with DOCS') {
                    steps {
                        unstash (name: 'prepped')
                        sh './configure --enable-drafts=yes --with-docs=yes'
                        sh 'make -k -j4 || make'
                        sh 'echo "Are GitIgnores good after make with docs? (should have no output below)"; git status -s || true'
                    }
                }
            }
        }
        stage ('check') {
            parallel {
                stage ('check with DRAFT') {
                    steps {
                        unstash (name: 'built-draft')
                        timeout (time: 5, unit: 'MINUTES') {
                            sh 'make check'
                        }
                        sh 'echo "Are GitIgnores good after make check with drafts? (should have no output below)"; git status -s || true'
                    }
                }
                stage ('check without DRAFT') {
                    steps {
                        unstash (name: 'built-nondraft')
                        timeout (time: 5, unit: 'MINUTES') {
                            sh 'make check'
                        }
                        sh 'echo "Are GitIgnores good after make check without drafts? (should have no output below)"; git status -s || true'
                    }
                }
            }
        }
        stage ('memcheck') {
            parallel {
                stage ('memcheck with DRAFT') {
                    steps {
                        unstash (name: 'built-draft')
                        timeout (time: 5, unit: 'MINUTES') {
                            sh 'make memcheck && exit 0 ; echo "Re-running failed ($?) memcheck with greater verbosity" >&2 ; make VERBOSE=1 memcheck-verbose'
                        }
                        sh 'echo "Are GitIgnores good after make memcheck with drafts? (should have no output below)"; git status -s || true'
                    }
                }
                stage ('memcheck without DRAFT') {
                    steps {
                        unstash (name: 'built-nondraft')
                        timeout (time: 5, unit: 'MINUTES') {
                            sh 'make memcheck && exit 0 ; echo "Re-running failed ($?) memcheck with greater verbosity" >&2 ; make VERBOSE=1 memcheck-verbose'
                        }
                        sh 'echo "Are GitIgnores good after make memcheck without drafts? (should have no output below)"; git status -s || true'
                    }
                }
            }
        }
        stage ('distcheck') {
            parallel {
                stage ('distcheck with DRAFT') {
                    steps {
                        unstash (name: 'built-draft')
                        timeout (time: 10, unit: 'MINUTES') {
                            sh 'make distcheck'
                        }
                        sh 'echo "Are GitIgnores good after make distcheck with drafts? (should have no output below)"; git status -s || true'
                    }
                }
                stage ('distcheck without DRAFT') {
                    steps {
                        unstash (name: 'built-nondraft')
                        timeout (time: 10, unit: 'MINUTES') {
                            sh 'make distcheck'
                        }
                        sh 'echo "Are GitIgnores good after make distcheck without drafts? (should have no output below)"; git status -s || true'
                    }
                }
            }
        }
    }
    post {
        success {
            script {
                if (currentBuild.getPreviousBuild()?.result != 'SUCCESS') {
                    // Uncomment desired notification

                    //slackSend (color: "#008800", message: "Build ${env.JOB_NAME} is back to normal.")
                    //emailext (to: "qa@example.com", subject: "Build ${env.JOB_NAME} is back to normal.", body: "Build ${env.JOB_NAME} is back to normal.")
                }
            }
        }
        failure {
            // Uncomment desired notification
            // Section must not be empty, you can delete the sleep once you set notification
            sleep 1
            //slackSend (color: "#AA0000", message: "Build ${env.BUILD_NUMBER} of ${env.JOB_NAME} ${currentBuild.result} (<${env.BUILD_URL}|Open>)")
            //emailext (to: "qa@example.com", subject: "Build ${env.JOB_NAME} failed!", body: "Build ${env.BUILD_NUMBER} of ${env.JOB_NAME} ${currentBuild.result}\nSee ${env.BUILD_URL}")
        }
    }
}
