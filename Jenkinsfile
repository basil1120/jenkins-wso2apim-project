pipeline {
    agent any

    environment {

        WSO2_APIM_HOST_DEV = "https://localhost:9443"
        WSO2_APIM_HOST_SIT = "https://localhost:9443"
        WSO2_APIM_HOST_UAT = "https://localhost:9443"
        WSO2_APIM_HOST_PRD = "https://localhost:9443"
        WSO2_TENANT    = "carbon.super"  // Change if using a tenant
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
                        apictl add env uat --apim ${WSO2_APIM_HOST_UAT}
                        apictl add env prd --apim ${WSO2_APIM_HOST_PRD}
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
}
