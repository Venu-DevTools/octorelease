pipeline {
    agent any

    environment {
        OCTOPUS_SERVER = 'https://devtools.octopus.app'
        SPACE_ID       = 'Spaces-162'
        PROJECT_NAME   = 'omega-alpha'
    }

    stages {
        stage('Login to Octopus') {
            when {
                branch 'release'
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

        stage('Sync BUILD_NUMBER Variable') {
            when {
                branch 'release'
            }
            steps {
                withCredentials([string(credentialsId: 'octopus-api-key', variable: 'OCTOPUS_API_KEY')]) {
                    sh '''
                        set -e

                        # Ensure jq is available
                        if ! command -v jq >/dev/null 2>&1; then
                            echo "Error: jq is not installed. Please install jq on your Jenkins agent."
                            exit 1
                        fi

                        # 1. List all variables in JSON
                        VAR_JSON=$(octopus project variables list \
                            --project "$PROJECT_NAME" \
                            --space "$SPACE_ID" \
                            --output-format json)

                        # 2. Extract IDs for all BUILD_NUMBER variables
                        VARIABLE_IDS=$(echo "$VAR_JSON" | jq -r '.[] | select(.Name == "BUILD_NUMBER") | .Id')

                        # 3. Delete each found BUILD_NUMBER variable
                        if [ -n "$VARIABLE_IDS" ]; then
                            echo "Found existing BUILD_NUMBER variable(s):"
                            for ID in $VARIABLE_IDS; do
                                echo " → deleting id=$ID"
                                octopus project variables delete BUILD_NUMBER \
                                    --name BUILD_NUMBER \
                                    --project "$PROJECT_NAME" \
                                    --space "$SPACE_ID" \
                                    --id "$ID" \
                                    --confirm
                            done
                        else
                            echo "No existing BUILD_NUMBER variable found."
                        fi

                        # 4. Create a fresh BUILD_NUMBER variable
                        echo "Creating BUILD_NUMBER = $BUILD_NUMBER"
                        octopus project variables create \\
                            --project "$PROJECT_NAME" \\
                            --space "$SPACE_ID" \\
                            --name "BUILD_NUMBER" \\
                            --value "$BUILD_NUMBER" \\
                            --type text
                    '''
                }
            }
        }

        // Dummy stage without any when condition
        stage('Dummy Stage') {
            steps {
                echo 'This dummy stage runs unconditionally, regardless of the branch.'
            }
        }
    }
}
