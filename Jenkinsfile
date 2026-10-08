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
    description: 'Mandatory for node updates, e.g. t3.large'
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
                    error("INSTANCE_TYPE is mandatory for node updates")
                }
            }

            echo "All mandatory parameters validated successfully" 
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
    }
}
