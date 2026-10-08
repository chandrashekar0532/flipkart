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
