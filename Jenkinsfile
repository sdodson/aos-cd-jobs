
properties( [
    [$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator', artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '60']],
    disableConcurrentBuilds(),
    [$class: 'HudsonNotificationProperty', enabled: false],
    [$class: 'RebuildSettings', autoRebuild: false, rebuildDisabled: false],
    [$class: 'ThrottleJobProperty', categories: [], limitOneJobWithMatchingParams: false, maxConcurrentPerNode: 0, maxConcurrentTotal: 0, paramsToUseForLimit: '', throttleEnabled: false, throttleOption: 'project'],
    pipelineTriggers([[$class: 'TimerTrigger', spec: '0 4 * * 2,4']])] )

build job: '../aos-cd-builds/build%2Fose',
    parameters: [   [$class: 'StringParameterValue', name: 'OSE_MAJOR', value: '3'],
                    [$class: 'StringParameterValue', name: 'OSE_MINOR', value: '4'],
                ]

build job: '../aos-cd-builds/build%2Fose',
    parameters: [   [$class: 'StringParameterValue', name: 'OSE_MAJOR', value: '3'],
                    [$class: 'StringParameterValue', name: 'OSE_MINOR', value: '3'],
                ]

build job: '../aos-cd-builds/build%2Fose',
    parameters: [   [$class: 'StringParameterValue', name: 'OSE_MAJOR', value: '3'],
                    [$class: 'StringParameterValue', name: 'OSE_MINOR', value: '2'],
                ]
