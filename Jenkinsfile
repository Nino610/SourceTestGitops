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
                REM Kiểm tra và xóa thư mục ${helmRepo} nếu đã tồn tại
                if exist ${helmRepo} rmdir /s /q ${helmRepo}

                REM Clone repo cấu hình từ GitHub
                git clone ${appConfigRepo} --branch ${appConfigBranch}
                cd ${helmRepo}

                REM Kiểm tra sự tồn tại của tệp app-demo-value.yaml trong thư mục app-demo
                if exist app-demo/app-demo-value.yaml (
                    REM Nếu tệp tồn tại, thay thế giá trị tag
                    powershell -Command "(Get-Content app-demo/app-demo-value.yaml) -replace '  tag: .*', '  tag: \"${version}\"' | Set-Content app-demo/app-demo-value.yaml"
                ) else (
                    REM Nếu tệp không tồn tại, thông báo và dừng pipeline
                    echo Tệp app-demo/app-demo-value.yaml không tồn tại!
                    exit /b 1
                )

                REM Thêm các thay đổi vào git, commit và push
                git add .
                git commit -m "Update to version ${version}"

                REM Pull trước khi push để tránh lỗi đồng bộ
                git pull origin ${appConfigBranch}

                REM Push lên remote
                git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/Nino610/ConfigTestGitops.git

                cd ..
                REM Xóa thư mục ${helmRepo} sau khi sử dụng
                if exist ${helmRepo} rmdir /s /q ${helmRepo}
            """
        }
    }
}

        }
    }
