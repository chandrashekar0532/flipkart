
pipeline {
    agent any

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'stage', 'uat', 'prod'],
            description: 'Select environment'
        )

        choice(
            name: 'CONFIG_TYPE',
            choices: ['both', 'environment', 'node'],
            description: 'Select configuration type'
        )

        string(
            name: 'INSTANCE_TYPE',
            defaultValue: '',
            description: 'Mandatory for node updates'
        )

        string(
            name: 'K8S_VERSION',
            defaultValue: '',
            description: 'Optional: Kubernetes version'
        )

        string(
            name: 'CPU',
            defaultValue: '',
            description: 'Optional: CPU value'
        )

        string(
            name: 'MEMORY',
            defaultValue: '',
            description: 'Optional: Memory value'
        )

string(
    name: 'ENV_APPNAME',
    defaultValue: '',
    description: 'Application name (mandatory for environment updates)'
)

string(
    name: 'ENV_VERSION',
    defaultValue: '',
    description: 'Application version, e.g. 2.0.0 (mandatory)'
)

string(
    name: 'ENV_REPLICAS',
    defaultValue: '',
    description: 'Number of replicas, e.g. 2 (mandatory)'
)

choice(
    name: 'ENV_LOGLEVEL',
    choices: ['KEEP', 'DEBUG', 'INFO', 'WARN', 'ERROR'],
    description: 'Optional: Select log level or KEEP existing value'
)

        
    }
    

    stages {
        stage('Display Parameters') {
            steps {
                echo "Environment: ${params.ENVIRONMENT}"
                echo "Config Type: ${params.CONFIG_TYPE}"
            }
        }

        stage('Validate Parameters') {
            steps {
                script {
                    if (!params.ENVIRONMENT?.trim()) {
                        error('ENVIRONMENT is mandatory')
                    }

                    if (!params.CONFIG_TYPE?.trim()) {
                        error('CONFIG_TYPE is mandatory')
                    }

                    if (params.CONFIG_TYPE in ['node', 'both']) {
                        if (!params.INSTANCE_TYPE?.trim()) {
                            error('INSTANCE_TYPE is mandatory for node updates')
                        }
                        
if (params.CONFIG_TYPE in ['environment', 'both']) {

    if (!params.ENV_APPNAME?.trim()) {
        error('ENV_APPNAME is mandatory for environment updates')
    }

    if (!params.ENV_VERSION?.trim()) {
        error('ENV_VERSION is mandatory for environment updates')
    }

    if (!(params.ENV_VERSION.trim() ==~ /\d+\.\d+\.\d+/)) {
        error('ENV_VERSION must follow format 1.0.0')
    }

    if (!params.ENV_REPLICAS?.trim()) {
        error('ENV_REPLICAS is mandatory for environment updates')
    }

    if (!(params.ENV_REPLICAS.trim() ==~ /[1-9][0-9]*/)) {
        error('ENV_REPLICAS must be a positive integer')
    }
}

                    }

                    echo 'All mandatory parameters validated successfully'
                }
            }
        }

        stage('Read Configuration Files') {
            steps {
                script {
                    def types = params.CONFIG_TYPE == 'both'
                        ? ['environment', 'node']
                        : [params.CONFIG_TYPE]

                    types.each { type ->
                        def path = "${type}/${params.ENVIRONMENT}.json"

                        if (!fileExists(path)) {
                            error("Configuration file missing: ${path}")
                        }

                        def config = readJSON file: path, returnPojo: true

                        if (!(config instanceof Map)) {
                            error("Expected a JSON object in ${path}")
                        }

                        echo "Validated JSON file: ${path}"
                    }
                }
            }
        }

        stage('Update JSON Configuration') {
            steps {
                script {
                    def types = params.CONFIG_TYPE == 'both'
                        ? ['environment', 'node']
                        : [params.CONFIG_TYPE]

                    types.each { type ->
                        def path = "${type}/${params.ENVIRONMENT}.json"

                        def config = readJSON file: path, returnPojo: true
                        
if (type == 'environment') {

    config.appName = params.ENV_APPNAME.trim()

    config.version = params.ENV_VERSION.trim()

    config.replicas = params.ENV_REPLICAS.trim().toInteger()

    if (params.ENV_LOGLEVEL != 'KEEP') {
        config.logLevel = params.ENV_LOGLEVEL
    }

    echo "Updated application: ${config.appName}"
    echo "Updated version: ${config.version}"
    echo "Updated replicas: ${config.replicas}"
    echo "Updated log level: ${config.logLevel}"
}


                        if (type == 'node') {
                            config.instanceType = params.INSTANCE_TYPE.trim()

                            if (params.K8S_VERSION?.trim()) {
                                config.kubernetes.version =
                                    params.K8S_VERSION.trim()
                            }

                            if (params.CPU?.trim()) {
                                config.resources.cpu = params.CPU.trim()
                            }

                            if (params.MEMORY?.trim()) {
                                config.resources.memory =
                                    params.MEMORY.trim()
                            }

                            writeJSON file: path,
                                      json: config,
                                      pretty: 4

                            echo "Updated local configuration: ${path}"
                        }
                    }
                }
            }
        }

        stage('Verify Updated JSON') {
            steps {
                script {
                    if (params.CONFIG_TYPE in ['node', 'both']) {
                        def path = "node/${params.ENVIRONMENT}.json"

                        def config = readJSON file: path, returnPojo: true

                        echo "Updated Instance Type: ${config.instanceType}"
                        echo "Updated Kubernetes Version: ${config.kubernetes.version}"
                        echo "Updated CPU: ${config.resources.cpu}"
                        echo "Updated Memory: ${config.resources.memory}"
                    }
                }
            }
       }
        stage('Create Git Feature Branch') {
    steps {
        script {
            def branchName =
                "feature/update-${params.ENVIRONMENT}-${env.BUILD_NUMBER}"

            env.FEATURE_BRANCH = branchName

            sh """
                git config user.name "Jenkins Automation"
                git config user.email "jenkins-automation@example.com"

                git switch -c "${branchName}"

                git add environment/*.json node/*.json

                if git diff --cached --quiet; then
                    echo "No configuration changes detected"
                else
                    git commit -m "Update ${params.ENVIRONMENT} configuration"
                fi
            """

            echo "Feature branch prepared: ${branchName}"
        }
    }
}
        stage('Push Feature Branch') {
    steps {
        withCredentials([
            gitUsernamePassword(
                credentialsId: 'github-private-creds',
                gitToolName: 'Default'
            )
        ]) {
            sh 'git push -u origin "$FEATURE_BRANCH"'
        }
    }
}
    
stage('Create GitHub Pull Request') {
    steps {
        withCredentials([
            usernamePassword(
                credentialsId: 'github-private-creds',
                usernameVariable: 'GITHUB_USER',
                passwordVariable: 'GITHUB_TOKEN'
            )
        ]) {
            sh '''
                set +x

                python3 - <<'PY'
import json
import os
import urllib.request

repo = "chandrashekar0532/flipkart"
branch = os.environ["FEATURE_BRANCH"]
token = os.environ["GITHUB_TOKEN"]

payload = {
    "title": f"Update configuration: {branch}",
    "head": branch,
    "base": "main",
    "body": "Automated configuration update by Jenkins."
}

request = urllib.request.Request(
    f"https://api.github.com/repos/{repo}/pulls",
    data=json.dumps(payload).encode(),
    headers={
        "Authorization": f"Bearer {token}",
        "Accept": "application/vnd.github+json",
        "X-GitHub-Api-Version": "2022-11-28",
        "Content-Type": "application/json"
    },
    method="POST"
)

with urllib.request.urlopen(request, timeout=30) as response:
    result = json.load(response)
    print("Pull Request created:", result["html_url"])
PY
            '''
        }
    }
}
    
    }
    
}
