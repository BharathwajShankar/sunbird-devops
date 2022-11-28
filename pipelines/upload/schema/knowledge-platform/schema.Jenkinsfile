@Library('deploy-conf') _
node() {
    try {
        String ANSI_GREEN = "\u001B[32m"
        String ANSI_NORMAL = "\u001B[0m"
        String ANSI_BOLD = "\u001B[1m"
        String ANSI_RED = "\u001B[31m"
        String ANSI_YELLOW = "\u001B[33m"

        stage('checkout public repo') {
            folder = new File("$WORKSPACE/.git")
            if (folder.exists())
            {
               println "Found .git folder. Clearing it.."
               sh'git clean -fxd'
            }
            checkout scm
        }

            ansiColor('xterm') {
                values = [:]
                currentWs = sh(returnStdout: true, script: 'pwd').trim()
                envDir = sh(returnStdout: true, script: "echo $JOB_NAME").split('/')[-3].trim()
                module = sh(returnStdout: true, script: "echo $JOB_NAME").split('/')[-2].trim()
                jobName = sh(returnStdout: true, script: "echo $JOB_NAME").split('/')[-1].trim()
                stage('deploy artifact'){
                    sh """
                rm -rf schema_repo
                git clone ${params.schema_repo} -b ${params.schema_repo_branch_or_tag} schema_repo
                    """
                    ansiblePlaybook = "${currentWs}/ansible/kp_upload-schema.yml"
                    ansibleExtraArgs = "--extra-vars \" source_name=${params.source_folder} source_path=${currentWs}/schema_repo/schemas \" --vault-password-file /var/lib/jenkins/secrets/vault-pass"  
                    values.put('currentWs', currentWs)
                    values.put('env', envDir)
                    values.put('module', module)
                    values.put('jobName', jobName)
                    values.put('ansiblePlaybook', ansiblePlaybook)
                    values.put('ansibleExtraArgs', ansibleExtraArgs)
                    println values
                    ansible_playbook_run(values)
                    currentBuild.result = 'SUCCESS'
                    currentBuild.description = "Private: ${params.private_branch}, Public: ${params.branch_or_tag}, schema_repo_branch_or_tag: ${params.schema_repo_branch_or_tag}"
                }
            }
        summary()
        }
    catch (err) {
        currentBuild.result = 'FAILURE'
        throw err
    }    
    finally {
        slack_notify(currentBuild.result)
        email_notify()
    }
}