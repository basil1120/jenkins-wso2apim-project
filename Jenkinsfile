pipeline {
    agent any

    environment {

        WSO2_TENANT    = "carbon.super"  // Change if using a tenant
        WSO2_MI_HOST_DEV = "https://localhost:8253"
        WSO2_MI_HOST_SIT = "https://localhost:8253"
        WSO2_APIM_HOST_DEV = "https://localhost:9443"
        WSO2_APIM_HOST_SIT = "https://localhost:9443"
        WSO2_PUBLISHER_HOST_DEV ="https://localhost:9443/publisher"
        WSO2_PUBLISHER_HOST_SIT ="https://localhost:9443/publisher"
        WSO2_DEVPORTAL_HOST_DEV ="https://localhost:9443/devportal"
        WSO2_DEVPORTAL_HOST_SIT ="https://localhost:9443/devportal"
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

        // stage('Clean & Remove APICTL Environments') {
        //     steps {
        //         script {
        //             sh """
        //             echo "---------- Listing APICTL Environments ---------"
        //             apictl get envs > apictl_envs.txt

        //             echo "---------- Removing APICTL Environments ---------"
        //             while read -r envName _; do
        //                 if [ "\$envName" != "NAME" ]; then
        //                     echo "Removing APICTL environment: \$envName"
        //                     apictl remove env "\$envName"
        //                 fi
        //             done < <(tail -n +2 apictl_envs.txt)
        //             """
        //         }
        //     }
        // }


        stage('Login to WSO2 API Manager') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'WSO2_CREDENTIALS', usernameVariable: 'WSO2_USERNAME', passwordVariable: 'WSO2_PASSWORD')]) {
                        sh """
                        echo "---------- Starting Setting APICTL Configurations ---------"
                        apictl set --http-request-timeout 90000
                        apictl set --tls-renegotiation-mode freely
                        echo "---------- Starting Setting APICTL Environments ---------"
                        apictl add env dev --apim ${WSO2_APIM_HOST_DEV}
                        apictl add env sit --apim ${WSO2_APIM_HOST_SIT}
                        echo "---------- Start APICTL Login ---------"
                        apictl login dev -u ${WSO2_USERNAME} -p ${WSO2_PASSWORD} --insecure
                        echo "---------- Start Set Export Directory ---------"
                        pwd
                        ls -lrt
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