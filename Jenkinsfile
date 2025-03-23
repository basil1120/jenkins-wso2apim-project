pipeline {
    agent any

    environment {
        // WSO2 API Manager Hosts
        WSO2_APIM_HOST_DEV = "https://localhost:9443"
        WSO2_APIM_HOST_SIT = "https://localhost:9443"
        WSO2_APIM_HOST_UAT = "https://localhost:9443"
        WSO2_APIM_HOST_PRD = "https://localhost:9443"
        WSO2_TENANT = "carbon.super"
    }

    parameters {
        choice(name: 'ENVIRONMENT', choices: ['DEV', 'SIT', 'UAT', 'PRD'], description: 'Select API Manager Environment')
    }

    stages {
        stage('Install APICTL') {
            steps {
                script {
                    sh """
                    if ! command -v apictl &> /dev/null; then
                        echo "Installing APICTL..."
                        curl -LO https://github.com/wso2/product-apim-tooling/releases/download/v4.3.0/apictl-4.3.0-linux-x64.tar.gz
                        tar -xzf apictl-4.3.0-linux-x64.tar.gz
                        sudo mv apictl /usr/local/bin/
                        chmod +x /usr/local/bin/apictl
                    else
                        echo "APICTL already installed - Version:"
                        apictl version
                    fi
                    """
                }
            }
        }

        stage('Validate & Add APICTL Environment') {
            steps {
                script {
                    def apiEnv = params.ENVIRONMENT.toLowerCase()
                    def apiHost = ""

                    // Select API Manager Host based on the chosen environment
                    switch(apiEnv) {
                        case "dev":
                            apiHost = env.WSO2_APIM_HOST_DEV
                            break
                        case "sit":
                            apiHost = env.WSO2_APIM_HOST_SIT
                            break
                        case "uat":
                            apiHost = env.WSO2_APIM_HOST_UAT
                            break
                        case "prd":
                            apiHost = env.WSO2_APIM_HOST_PRD
                            break
                    }

                    sh """
                    echo "Checking if APICTL environment '${apiEnv}' already exists..."
                    apictl list env | grep -w '${apiEnv}'
                    if [ $? -eq 0 ]; then
                        echo "Environment '${apiEnv}' already exists. Skipping addition."
                    else
                        echo "Adding APICTL environment '${apiEnv}'..."
                        apictl add env ${apiEnv} --apim ${apiHost}
                    fi
                    """
                }
            }
        }

        stage('Validate & Login to WSO2 API Manager') {
            steps {
                script {
                    def apiEnv = params.ENVIRONMENT.toLowerCase()

                    withCredentials([usernamePassword(credentialsId: 'WSO2_CREDENTIALS', usernameVariable: 'WSO2_USERNAME', passwordVariable: 'WSO2_PASSWORD')]) {
                        sh """
                        echo "Checking if already logged into '${apiEnv}'..."
                        apictl get apis -e ${apiEnv} --insecure > /dev/null 2>&1
                        if [ $? -eq 0 ]; then
                            echo "Already logged into '${apiEnv}'. Skipping login."
                        else
                            echo "Logging into WSO2 API Manager ('${apiEnv}')..."
                            apictl login ${apiEnv} -u ${WSO2_USERNAME} -p ${WSO2_PASSWORD} --insecure
                        fi
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                def subject = "✅ SUCCESS: WSO2 APICTL Login & Setup Completed for ${params.ENVIRONMENT}"
                def body = """<p>The Jenkins pipeline has completed successfully.</p>
                              <p><strong>Environment:</strong> ${params.ENVIRONMENT}</p>
                              <p><a href="${env.BUILD_URL}console">View Console Logs</a></p>"""

                emailext (
                    to: "basiljereh@gmail.com",
                    subject: subject,
                    body: body,
                    recipientProviders: [[$class: 'DevelopersRecipientProvider']]
                )
            }
        }

        failure {
            script {
                def subject = "❌ FAILURE: WSO2 APICTL Login & Setup Failed for ${params.ENVIRONMENT}"
                def body = """<p>The Jenkins pipeline has failed.</p>
                              <p><strong>Environment:</strong> ${params.ENVIRONMENT}</p>
                              <p><a href="${env.BUILD_URL}console">View Console Logs</a></p>"""

                emailext (
                    to: "basiljereh@gmail.com",
                    subject: subject,
                    body: body,
                    recipientProviders: [[$class: 'DevelopersRecipientProvider']]
                )
            }
        }
    }
}
