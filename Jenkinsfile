#!/usr/bin/env groovy

// https://issues.jenkins-ci.org/browse/JENKINS-33511
def set_workspace() {
    if(env.WORKSPACE == null) {
        env.WORKSPACE = pwd()
    }
}

def version(f) {
    def matcher = readFile(f) =~ /Version:\s+([.0-9]+)/
    matcher ? matcher[0][1] : null
}

def mail_success() {
    mail(
        to: "${MAIL_LIST_SUCCESS}",
        replyTo: 'tdawson@redhat.com',
        subject: "Images have been refreshed",
        body: """\
Jenkins job: ${env.BUILD_URL}
${OSE_MAJOR}.${OSE_MINOR}, Group:${OSE_GROUP}, Repo:${OSE_REPO}
""");
}

node('buildvm-devops') {

    // Expose properties for a parameterized build
    properties(
            [[$class              : 'ParametersDefinitionProperty',
              parameterDefinitions:
                      [
                              [$class: 'hudson.model.ChoiceParameterDefinition', choices: "3", defaultValue: '3', description: 'OSE Major Version', name: 'OSE_MAJOR'],
                              [$class: 'hudson.model.ChoiceParameterDefinition', choices: "1\n2\n3\n4\n5\n6\n7", defaultValue: '4', description: 'OSE Minor Version', name: 'OSE_MINOR'],
                              [$class: 'hudson.model.ChoiceParameterDefinition', choices: "base\nmetrics\nlogging\njenkins\nmisc\nall", defaultValue: 'base', description: 'Which group to refresh', name: 'OSE_GROUP'],
                              [$class: 'hudson.model.ChoiceParameterDefinition', choices: "http://file.rdu.redhat.com/~tdawson/repo/aos-unsigned-building.repo\nhttp://file.rdu.redhat.com/~tdawson/repo/aos-unsigned-errata-building.repo\nhttp://file.rdu.redhat.com/~tdawson/repo/aos-signed-building.repo", defaultValue: 'http://file.rdu.redhat.com/~tdawson/repo/aos-unsigned-building.repo', description: 'Which repo to use', name: 'OSE_REPO'],
                              [$class: 'hudson.model.StringParameterDefinition', defaultValue: 'tdawson@redhat.com', description: 'Success Mailing List', name: 'MAIL_LIST_SUCCESS'],
                              [$class: 'hudson.model.StringParameterDefinition', defaultValue: 'tdawson@redhat.com', description: 'Failure Mailing List', name: 'MAIL_LIST_FAILURE'],
                      ]
             ]]
    )
    
    // Force Jenkins to fail early if this is the first time this job has been run/and or new parameters have not been discovered.
    echo "${OSE_MAJOR}.${OSE_MINOR}, Group:${OSE_GROUP}, Repo:${OSE_REPO} MAIL_LIST_SUCCESS:[${MAIL_LIST_SUCCESS}], MAIL_LIST_FAILURE:[${MAIL_LIST_FAILURE}]"

    set_workspace()
    stage('Refresh Images') {
        try {
            checkout scm

            checkout changelog: false, poll: false,
                       scm: [$class: 'GitSCM', branches: [[name: 'build-scripts']],
                                doGenerateSubmoduleConfigurations: false,
                                extensions: [   [$class: 'RelativeTargetDirectory', relativeTargetDir: 'build-scripts'],
                                                [$class: 'WipeWorkspace']], submoduleCfg: [],
                                                userRemoteConfigs: [[url: 'https://github.com/openshift/aos-cd-jobs.git']]]

            env.PATH = "${pwd()}/build-scripts/:${env.PATH}"

            sshagent(['openshift-bot']) { // merge-and-build must run with the permissions of openshift-bot to succeed
                sh "ose_images.sh update_docker --bump_release --force --branch rhaos-${OSE_MAJOR}.${OSE_MINOR}-rhel-7 --group ${OSE_GROUP}"
                sh "ose_images.sh build --branch rhaos-${OSE_MAJOR}.${OSE_MINOR}-rhel-7 --group ${OSE_GROUP} --repo ${OSE_REPO}"
            }

            // Replace flow control with: https://jenkins.io/blog/2016/12/19/declarative-pipeline-beta/ when available
            mail_success()


        } catch ( err ) {
            // Replace flow control with: https://jenkins.io/blog/2016/12/19/declarative-pipeline-beta/ when available
            mail(to: "${MAIL_LIST_FAILURE}",
                    subject: "Error Refreshing Images: ${OSE_MAJOR}.${OSE_MINOR} ${OSE_GROUP} ${OSE_REPO}",
                    body: """Encoutered an error while running merge-and-build.sh: ${err}


Jenkins job: ${env.BUILD_URL}
""");
            // Re-throw the error in order to fail the job
            throw err
        }

    }
}
