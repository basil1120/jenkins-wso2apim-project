pipeline {
    agent any

    environment {
        WSO2_APIM_HOST = "https://localhost:9443"  // Change to your API Manager URL
        WSO2_TENANT    = "carbon.super"  // Change if using a tenant
        // WSO2_USERNAME  = credentials('WSO2_USERNAME')  // Jenkins credentials ID for username
        // WSO2_PASSWORD  = credentials('WSO2_PASSWORD')  // Jenkins credentials ID for password
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
                        apictl login ${WSO2_APIM_HOST} -u ${WSO2_USERNAME} -p ${WSO2_PASSWORD} --insecure
                        """
                    }
                }
            }
        }

        // stage('Login to WSO2 API Manager') {
        //     steps {
        //         script {
        //             withCredentials([usernamePassword(credentialsId: 'WSO2_CREDENTIALS', usernameVariable: 'WSO2_USERNAME', passwordVariable: 'WSO2_PASSWORD')]) {
        //                 sh """
        //                 apictl login ${WSO2_APIM_HOST} -u ${WSO2_USERNAME} -p ${WSO2_PASSWORD} -t ${WSO2_TENANT} --insecure
        //                 """
        //             }
        //         }
        //     }
        // }

    }
}
