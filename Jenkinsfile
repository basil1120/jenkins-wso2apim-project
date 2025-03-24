pipeline {
    agent any

    environment {

        WSO2_TENANT    = "carbon.super"  // Change if using a tenant
        //MI Host
        WSO2_MI_HOST_DEV = "https://localhost:8253"
        WSO2_MI_HOST_SIT = "https://localhost:8253"
        //API Manager Host
        WSO2_APIM_HOST_DEV = "https://localhost:9443"
        WSO2_APIM_HOST_SIT = "https://localhost:9443"
        WSO2_APIM_HOST_CLOUD = "https://api.mwnmarketplace.com:30532"
        //Publisher Host
        WSO2_PUBLISHER_HOST_DEV ="https://localhost:9443/publisher"
        WSO2_PUBLISHER_HOST_SIT ="https://localhost:9443/publisher"
        //Dev-Portal Host
        WSO2_DEVPORTAL_HOST_DEV ="https://localhost:9443/devportal"
        WSO2_DEVPORTAL_HOST_SIT ="https://localhost:9443/devportal"
        //Admin-Portal Host
        WSO2_ADMINPORTAL_HOST_DEV ="https://localhost:9443/admin"
        WSO2_ADMINPORTAL_HOST_SIT ="https://localhost:9443/admin"

        // Select Environment (Set this as a parameter or in Jenkins UI)
        SELECTED_ENV = "DEV"

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
                        apictl version
                        echo "APICTL already installed"
                    fi
                    """
                }
            }
        }

        stage('Clean & Remove APICTL Environments') {
            steps {
                script {
                    sh """
                    echo "---------- Listing APICTL Environments ---------"
                    apictl get envs | tail -n +2 | awk '{print \$1}' > apictl_envs.txt

                    echo "---------- Removing APICTL Environments ---------"
                    while read -r envName; do
                        if [ ! -z "\$envName" ]; then
                            echo "Removing APICTL environment: \$envName"
                            apictl remove env "\$envName"
                        fi
                    done < apictl_envs.txt
                    """
                }
            }
        }

        stage('Add / Setting APICTL Environments') {
            steps {
                script {
                    sh """
                    echo "---------- Starting Setting APICTL Environments ----------"
                    apictl set --http-request-timeout 90000
                    apictl set --tls-renegotiation-mode freely
                    apictl add env dev --apim ${WSO2_APIM_HOST_DEV} --admin ${WSO2_ADMINPORTAL_HOST_DEV} --publisher ${WSO2_PUBLISHER_HOST_DEV} --devportal ${WSO2_DEVPORTAL_HOST_DEV} --mi ${WSO2_MI_HOST_DEV}
                    apictl add env sit --apim ${WSO2_APIM_HOST_SIT}
                    apictl add env cloud --apim ${WSO2_APIM_HOST_CLOUD}
                    """
                }
            }
        }

        stage('Export APIs from WSO2 API Manager Source') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'WSO2_CREDENTIALS', usernameVariable: 'WSO2_USERNAME', passwordVariable: 'WSO2_PASSWORD')]) {
                        sh """
                        echo "---------- Starting Login To Source Environment & Exporting ----------"
                        apictl login cloud -u ${WSO2_USERNAME} -p ${WSO2_PASSWORD} --insecure
                        apictl set --export-directory /Users/basam/.wso2apictl/exported
                        apictl get apis -e cloud --insecure
                        apictl export apis -e cloud --insecure
                        apictl logout cloud --insecure
                        """
                    }
                }
            }
        }

        stage('Import APIs to WSO2 API Manager Destination') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'WSO2_CREDENTIALS', usernameVariable: 'WSO2_USERNAME', passwordVariable: 'WSO2_PASSWORD')]) {
                        sh """
                        echo "---------- Starting Login To Destination Environment & Importing ----------"
                        apictl login dev -u ${WSO2_USERNAME} -p ${WSO2_PASSWORD} --insecure
                        echo "---------- Importing Exported APIs ----------"
                        EXPORT_DIRX="/Users/basam/.wso2apictl/exported"
                        EXPORT_DIR="/Users/basam/.wso2apictl/exported/migration/cloud/tenant-default/apis"

                        for api_zip in "\$EXPORT_DIR"/*.zip; do
                            if [[ -f "\$api_zip" ]]; then
                                echo "Importing API: \$api_zip..."
                                apictl import api -f "\$api_zip" -e dev --insecure
                                echo "------------------------------"
                            else
                                echo "No ZIP files found in \$EXPORT_DIR"
                            fi
                        done

                        echo "✅ API import process completed!"

                        echo "---------- Logging out from Destination Environment (dev) -----------"
                        apictl logout dev --insecure
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