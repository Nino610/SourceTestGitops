//pull config repo ok
// update tag in config repo ok
// not yet commit and push
def appSourceRepo = 'https://github.com/Nino610/SourceTestGitops.git'
def appSourceBranch = 'main'

def appConfigRepo = 'https://github.com/Nino610/ConfigTestGitops.git'
def appConfigBranch = 'main'
def helmRepo = "app-helmchart"
def helmChart = "app-demo"
def helmValueFile = "app-demo/values.yaml"

def dockerhubAccount = 'dockerhub'
def githubAccount = 'github'

dockerBuildCommand = './'
def version = "v1.${BUILD_NUMBER}"

pipeline {
    agent any    
   
    environment {
        DOCKER_REGISTRY = 'https://registry-1.docker.io'
        DOCKER_IMAGE_NAME = "anhdt172/test-gitops"
        DOCKER_IMAGE = "registry-1.docker.io/${DOCKER_IMAGE_NAME}"
    }

    stages {        
        stage('Checkout project') {
          steps {
            git branch: appSourceBranch,
                credentialsId: githubAccount,
                url: appSourceRepo
          }
        }
        stage('Build And Push Docker Image') {
            steps {
                script {
                    bat "git reset --hard"
                    bat "git clean -f"                    
					app = docker.build(DOCKER_IMAGE_NAME, dockerBuildCommand)
                    docker.withRegistry( DOCKER_REGISTRY, dockerhubAccount ) {
                       app.push(version)
                    }
                    bat "docker rmi ${DOCKER_IMAGE_NAME} -f"
                    bat "docker rmi ${DOCKER_IMAGE}:${version} -f"
                    echo "build và push xong rồi"
                }
            }
        }

stage('Update value in helm-chart') {
    steps {
        withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
            script {
                // Remove existing helm repo if exists
                sh """
                    [[ -d ${helmRepo} ]] && rm -r ${helmRepo}
                    git clone ${appConfigRepo} --branch ${appConfigBranch}
                    cd ${helmRepo}

                    // Update the tag in values.yaml file
                    sed -i 's|  tag: .*|  tag: "${version}"|' ${helmValueFile}

                    // Add changes and commit
                    git add .
                    git commit -m "Update to version ${version}"

                    // Fetch the latest changes from remote repository
                    git fetch origin ${appConfigBranch}

                    // Merge the fetched changes into your local branch
                    git merge origin/${appConfigBranch}

                    // Push the changes to remote repository
                    git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/Nino610/ConfigTestGitops.git
                    cd ..
                    
                    // Clean up by removing the helm repo
                    [[ -d ${helmRepo} ]] && rm -r ${helmRepo}
                """
            }
        }
    }
}


        }
    }
