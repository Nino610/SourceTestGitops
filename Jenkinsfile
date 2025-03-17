//pull config repo ok
// update tag in config repo ok
// not yet commit and push
def appSourceRepo = 'https://github.com/Nino610/SourceTestGitops'
def appSourceBranch = 'main'

def appConfigRepo = 'https://github.com/Nino610/ConfigTestGitops'
def appConfigBranch = 'main'
def helmRepo = "app-helmchart"
def helmChart = "app-demo"
def helmValueFile = "app-demo/app-demo-value.yaml"

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
    bat """
        if exist ${helmRepo} rmdir /s /q ${helmRepo}
        git clone ${appConfigRepo} --branch ${appConfigBranch}
        cd ${helmRepo}
        
        powershell -Command "(Get-Content ${helmValueFile}) -replace '  tag: .*', '  tag: \"${version}\"' | Set-Content ${helmValueFile}"
        
        git add . 
        git commit -m "Update to version ${version}"
        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/Nino610/ConfigTestGitops.git
        
        cd ..
        if exist ${helmRepo} rmdir /s /q ${helmRepo}
    """
}

				}				
            }
        }
    }
