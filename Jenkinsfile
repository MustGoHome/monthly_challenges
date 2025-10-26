pipeline {
    agent { label 'web' }

    environment {
        PROJECT_DIR = "/data/monthly_challenges"
        VENV_PATH = "${PROJECT_DIR}/venv_django_311"
        ALLOWED_HOSTS_LIST = "'127.0.0.1', 'localhost', '192.168.10.150'"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "🔹 Git 저장소에서 코드 가져오는 중..."
                checkout scm
            }
        }

        stage('Prepare Project Directory') {
            steps {
                echo "🔹 기존 프로젝트 디렉터리 정리 및 동기화..."
                sh '''
                    sudo rm -rf ${PROJECT_DIR}
                    sudo mkdir -p ${PROJECT_DIR}
                    sudo cp -r ./* ${PROJECT_DIR}/
                '''
            }
        }

        stage('Configure Settings') {
            steps {
                echo "🔹 settings.py 프로덕션 설정 적용 중..."
                sh '''
                    SETTINGS_FILE="${PROJECT_DIR}/monthly_challenges/settings.py"

                    sudo sed -i "s/^from pathlib import Path/from pathlib import Path, os/" \$SETTINGS_FILE
                    sudo sed -i "/^DEBUG = /c\\DEBUG = False" \$SETTINGS_FILE
                    sudo sed -i "/^ALLOWED_HOSTS/c\\ALLOWED_HOSTS = [${ALLOWED_HOSTS_LIST}]" \$SETTINGS_FILE
                    sudo sed -i "/^STATIC_URL = /c\\STATIC_URL = '/static/'" \$SETTINGS_FILE
                    echo "STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')" >> \$SETTINGS_FILE
                '''
            }
        }

        stage('Setup Virtual Environment') {
            steps {
                echo "🔹 Python 가상 환경 빌드 및 정적 파일 설정 중..."
                sh '''
                    cd ${PROJECT_DIR}
                    /usr/bin/python3.11 -m venv venv_django_311
                    source ${VENV_PATH}/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                    pip install mod_wsgi
                    python3.11 manage.py collectstatic --noinput
                    deactivate
                '''
            }
        }

        stage('Set Permissions') {
            steps {
                echo "🔹 디렉터리 권한 설정 중..."
                sh '''
                    sudo chown -R apache:apache /data
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Django 빌드 및 정적파일 수집 완료!"
        }
        failure {
            echo "❌ 빌드 실패: 로그를 확인하세요."
        }
    }
}
