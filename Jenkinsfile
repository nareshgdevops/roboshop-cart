pipeline {
    agent any

    stages {
        stage('NodeJS Build app dependencies') {
            when {
                expression { env.APP_TYPE == 'nodejs' }
            }
            steps {
                sh 'npm install'
            }
        }
        stage('Maven Build app dependencies') {
            when {
                expression { env.APP_TYPE == 'maven' }
            }
            steps {
                sh 'mvn package'
            }
        }
        stage('Python Build app dependencies') {
            when {
                expression { env.APP_TYPE == 'python' }
            }
            steps {
                sh 'echo python package'
            }
        }
        stage('Go lang Build app dependencies') {
            when {
                expression { env.APP_TYPE == 'golang' }
            }
            steps {
                sh 'go build'
            }
        }
        stage('Angular Build app dependencies') {
            when {
                expression { env.APP_TYPE == 'angular' }
            }
            steps {
                sh 'echo angular build'
            }
        }
        stage('Sonar Qube Scan') {
            when {
                expression { env.APP_TYPE != 'python' }
            }
            steps {
                sh 'echo /home/github/sonar-scanner-7.3.0.5189-linux-x64/bin/sonar-scanner -Dsonar.host.url=http://sonarqube.nareshdevops1218.online:9000 -Dsonar.token=$SONAR_TOKEN -Dsonar.projectKey=roboshop-${env.appName}'
            }
        }

        stage('CheckMarx SAST Scan') {
            steps {
                sh 'echo cx scan create -s . --project-name roboshop-${{ inputs.appName }} --scan-types sast'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    def secrets = [
                        [path: 'roboshop-infra/data/azure-service-priniciple', engineVersion: 1, secretValues: [
                            [envVar: 'AZURE_SUBSCRIPTION_ID', vaultKey: 'AZURE_SUBSCRIPTION_ID'],
                            [envVar: 'AZURE_CLIENT_ID', vaultKey: 'AZURE_CLIENT_ID'],
                            [envVar: 'AZURE_SECRET', vaultKey: 'AZURE_SECRET'],
                            [envVar: 'AZURE_TENANT', vaultKey: 'AZURE_TENANT']]],
                        [path: 'roboshop-infra/data/sonarqube', engineVersion: 2, secretValues: [
                            [envVar: 'SONAR_TOKEN', vaultKey: 'sonar_token']]]
                    ]
                    // inside this block your credentials will be available as env variables
                    withVault([vaultSecrets: secrets]) {
                        sh 'echo AZURE_SUBSCRIPTION_ID'
                        sh 'echo AZURE_CLIENT_ID'
                        sh 'echo AZURE_SECRET'
                        sh 'echo AZURE_TENANT'
                        sh 'SONAR_TOKEN'
                        sh "az login --service-principal --username $AZURE_CLIENT_ID --password $AZURE_SECRET --tenant $AZURE_TENANT"
                        sh 'az acr login --name nareshgdevops'
                        sh 'docker build -t nareshgdevops.azurecr.io/roboshop-$APP_NAME:$BUILD_NUMBER .'
                        echo 'trivy image nareshgdevops.azurecr.io/roboshop-${env.APP_NAME}:${{ github.sha }} --severity HIGH,CRITICAL --ignore-unfixed --exit-code 1'
                    }
                }
            }
        }

        stage('Docker Push') {
            steps {
                sh "az login --service-principal --username $AZURE_CLIENT_ID --password $AZURE_SECRET --tenant $AZURE_TENANT"
                sh 'az acr login --name nareshgdevops'
                sh 'docker push nareshgdevops.azurecr.io/roboshop-$APP_NAME:$BUILD_NUMBER'
            }
        }
    }
}

//      - name: Import Vault Secrets
//        if: inputs.docker_build == 'true'
//        id: import-secrets
//        uses: hashicorp/vault-action@v2
//        with:
//          url: http://vault.nareshdevops1218.online:8200/
//          token: ${{ secrets.VAULT_TOKEN }}
//          secrets: |
//            roboshop-infra/data/azure-service-priniciple AZURE_SUBSCRIPTION_ID | AZURE_SUBSCRIPTION_ID ;
//            roboshop-infra/data/azure-service-priniciple AZURE_CLIENT_ID | AZURE_CLIENT_ID ;
//            roboshop-infra/data/azure-service-priniciple AZURE_SECRET | AZURE_SECRET ;
//            roboshop-infra/data/azure-service-priniciple AZURE_TENANT | AZURE_TENANT ;
//            roboshop-infra/data/sonarqube sonar_token | SONAR_TOKEN ;
//
//      - name: Sonar Qube Scan - Java
//        if: inputs.docker_build == 'true' && inputs.appType == 'maven'
//        run: |
//          echo /home/github/sonar-scanner-7.3.0.5189-linux-x64/bin/sonar-scanner -Dsonar.host.url=http://sonarqube.nareshdevops1218.online:9000 -Dsonar.token=$SONAR_TOKEN -Dsonar.projectKey=roboshop-${{ inputs.appName }} -Dsonar.java.binaries=./target/
//          ### -Dsonar.qualitygate.wait=true
//          ### Above option to fail the pipeline, As of now we are not doing it.




//      - name: Docker Build
//        if: inputs.docker_build == 'true'
//        run: |
//          az login --service-principal --username $AZURE_CLIENT_ID --password $AZURE_SECRET --tenant $AZURE_TENANT
//          az acr login --name nareshgdevops
//          docker build -t nareshgdevops.azurecr.io/roboshop-${{ inputs.appName }}:${{ github.sha }} .
//          trivy image nareshgdevops.azurecr.io/roboshop-${{ inputs.appName }}:${{ github.sha }} --severity HIGH,CRITICAL --ignore-unfixed --exit-code 1
//
//      - name: Docker Push
//        if: inputs.docker_build == 'true'
//        run: |
//          az login --service-principal --username $AZURE_CLIENT_ID --password $AZURE_SECRET --tenant $AZURE_TENANT
//          az acr login --name nareshgdevops
//          docker push nareshgdevops.azurecr.io/roboshop-${{ inputs.appName }}:${{ github.sha }}
