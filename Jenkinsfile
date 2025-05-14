pipeline {
    agent any

    environment {
        OCTOPUS_SERVER = 'https://devtools.octopus.app'
        SPACE_ID       = 'Spaces-162'
        PROJECT_NAME   = 'omega-alpha'
    }

    stages {
        stage('Dummy Stage') {
            steps {
                echo "BRANCH_NAME: ${env.GIT_BRANCH}"
            }
        }

        stage('Login to Octopus') {
            when {
                expression { env.GIT_BRANCH == 'origin/release' }
            }
            steps {
                withCredentials([string(credentialsId: 'octopus-api-key', variable: 'OCTOPUS_API_KEY')]) {
                    sh """
                        octopus login \\
                            --server "$OCTOPUS_SERVER" \\
                            --api-key "$OCTOPUS_API_KEY"
                    """
                }
            }
        }

        stage('Reset BUILD_NUMBER Variable') {
            when {
                expression { env.GIT_BRANCH == 'origin/release' }
            }
            steps {
                withCredentials([string(credentialsId: 'octopus-api-key', variable: 'OCTOPUS_API_KEY')]) {
                    sh '''
                        set -e

                        if ! command -v jq >/dev/null 2>&1; then
                            echo "Error: jq is not installed."
                            exit 1
                        fi

                        echo "Fetching existing BUILD_NUMBER variables..."
                        VAR_JSON=$(octopus project variables list \
                            --project "$PROJECT_NAME" \
                            --space "$SPACE_ID" \
                            --output-format json)

                        VARIABLE_IDS=$(echo "$VAR_JSON" | jq -r '.[] | select(.Name == "BUILD_NUMBER") | .Id')

                        if [ -n "$VARIABLE_IDS" ]; then
                            echo "Deleting existing BUILD_NUMBER variables..."
                            for ID in $VARIABLE_IDS; do
                                echo " → deleting id=$ID"
                                octopus project variables delete BUILD_NUMBER \
                                    --name BUILD_NUMBER \
                                    --project "$PROJECT_NAME" \
                                    --space "$SPACE_ID" \
                                    --id "$ID" \
                                    --confirm
                            done

                            echo "Waiting for Octopus to release lock on variable set..."
                            sleep 5
                        fi

                        echo "Creating new BUILD_NUMBER = $BUILD_NUMBER"
                        octopus project variables create \
                            --project "$PROJECT_NAME" \
                            --space "$SPACE_ID" \
                            --name "BUILD_NUMBER" \
                            --value "$BUILD_NUMBER" \
                            --type text
                    '''
                }
            }
        }

        stage('Create Release in Octopus') {
            when {
                expression { env.GIT_BRANCH == 'origin/release' }
            }
            steps {
                withCredentials([string(credentialsId: 'octopus-api-key', variable: 'OCTOPUS_API_KEY')]) {
                    sh '''
                        set -e
                        echo "Creating release for project: $PROJECT_NAME using BUILD_NUMBER: $BUILD_NUMBER"

                        octopus release create \
                            --project "$PROJECT_NAME" \
                            --space "$SPACE_ID"
                    '''
                }
            }
        }
    }
}
