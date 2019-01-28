node('build-slave') {
    try {
        String ANSI_GREEN = "\u001B[32m"
        String ANSI_NORMAL = "\u001B[0m"
        String ANSI_BOLD = "\u001B[1m"
        String ANSI_RED = "\u001B[31m"
        String ANSI_YELLOW = "\u001B[33m"

        if (params.size() == 0){
            properties([[$class: 'RebuildSettings', autoRebuild: false, rebuildDisabled: false], parameters([string(defaultValue: '', description: '<font color=teal size=2>If you want to build from a tag, specify the tag name. If this parameter is blank, latest commit hash will be used to build</font>', name: 'tag', trim: false)])])

            ansiColor('xterm') {
                println (ANSI_BOLD + ANSI_GREEN + '''\
                        First run of the job. Parameters created. Stopping the current build.
                        Please trigger new build and provide parameters if required.
                        '''.stripIndent().replace("\n"," ") + ANSI_NORMAL)
            }
            return
        }

        ansiColor('xterm') {
            stage('Checkout') {
                cleanWs()
                if(params.tag == ""){
                    checkout scm
                    commit_hash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    branch_name = sh(script: 'git name-rev --name-only HEAD | rev | cut -d "/" -f1| rev', returnStdout: true).trim()
                    artifact_version = branch_name + "_" + commit_hash
                    println(ANSI_BOLD + ANSI_YELLOW + "Tag not specified, using the latest commit hash: " + commit_hash + ANSI_NORMAL)
                }
                else {
                    def scmVars = checkout scm
                    checkout scm: [$class: 'GitSCM', branches: [[name: "refs/tags/$params.tag"]],  userRemoteConfigs: [[url: scmVars.GIT_URL]]]
                    artifact_version = params.tag
                    println(ANSI_BOLD + ANSI_YELLOW + "Tag specified, building from tag: " + params.tag + ANSI_NORMAL)
                }
                echo "artifact_version: "+ artifact_version
            }
        }

        stage('Build') {
            sh """
            cd cassandra-migration
            mvn clean install -DskipTests
            """
        }


        stage('Archive artifacts'){
            sh """
                        mkdir cassandra_artifacts
                        cp cassandra-migration/target/cassandra-migration-*-jar-with-dependencies.jar cassandra_artifacts
                        zip -j cassandra_artifacts.zip:${artifact_version} cassandra_artifacts/*
                    """
            archiveArtifacts artifacts: "cassandra_artifacts.zip:${artifact_version}", fingerprint: true, onlyIfSuccessful: true
            sh """echo {\\"artifact_name\\" : \\"cassandra_artifacts.zip\\", \\"artifact_version\\" : \\"${artifact_version}\\", \\"node_name\\" : \\"${env.NODE_NAME}\\"} > metadata.json"""
            archiveArtifacts artifacts: 'metadata.json', onlyIfSuccessful: true
            currentBuild.description = "${artifact_version}"
        }
    }

    catch (err) {
        currentBuild.result = "FAILURE"
        throw err
    }

}
