pipeline {
    agent { label 'pipes-docker-agent' }
    environment {  
        // Azure Managed Identity
        //AZURE_MI = credentials('azure-mi')

        // use your Jenkins MI
        //JENKINS_USER_MI = 'a2711b6c-use-your-own-fa539d95817b'

        // Sandbox ACR
        REGISTRY_NAME = "gacr2nsandbox1"
        HELM_REGISTRY = "${REGISTRY_NAME}.azurecr.io"

        // Path to helm chart
        HELM_CHART = "demo-chart" 
        HELM_REPO = "gap/Pofremu/demo-helm-repo"
        SOURCE_FOLDER = "helm/${HELM_CHART}"
    }
    stages {
        stage('Init Tools') {
            steps {
                sh ('''
                    # Install homebrew
                    mkdir -p $HOME/.homebrew
                    curl -L https://github.com/Homebrew/brew/tarball/master | tar xz --strip 1 -C $HOME/.homebrew

                    eval "$($HOME/.homebrew/bin/brew shellenv)"
                    brew update --force --quiet
                    chmod -R go-w "$(brew --prefix)/share/zsh"
                ''')
            }
        } 
        stage('Lint Helm Chart') {            
            steps {
                echo 'Linting Chart...'
                sh ('''
                    # getting brew into the path
                    eval "$($HOME/.homebrew/bin/brew shellenv)"
                    brew update --force --quiet
                    chmod -R go-w "$(brew --prefix)/share/zsh"

                    # tools we will need 
                    brew install helm    

                    # Lint helm chart
                    helm lint ./helm/demo-chart
                ''') 
            }
        }
        stage('Reading Chart version') {
            steps{
                script {
                    env.HELM_TAG = sh (script: 
                       'eval "$($HOME/.homebrew/bin/brew shellenv)" && \
                        brew update --force --quiet && \
                        chmod -R go-w "$(brew --prefix)/share/zsh" && \
                        brew install yq  && \
                        HELM_TAG=$(yq e ".version" ${SOURCE_FOLDER}/Chart.yaml) && \
                        echo "${HELM_TAG}"',
                        returnStdout: true
                    ).trim()
                    println "The Chart version is ${HELM_TAG}"   
                }                
            }
        }
        stage('Install chart on sandbox') {
            steps{
                sh ('''
                    # getting brew into the path
                    eval "$($HOME/.homebrew/bin/brew shellenv)"
                    brew update --force --quiet
                    chmod -R go-w "$(brew --prefix)/share/zsh"

                    # tools we will need 
                    brew install az
                    brew install helm
                    brew install kubectl


                    # Login to azure
                    az login -u <username> -p <password>

                    # Set correct context
                    kubectl config use-context g-aks-2n-paas-sandbox-01-admin

                    # Install chart
                    helm install ./helm/demo-chart
                ''')
            }
        } 
    }
}