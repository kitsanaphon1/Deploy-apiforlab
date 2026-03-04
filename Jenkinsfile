pipeline {
    agent any

    environment {
        DOCKER_HOST  = "apiadmin@172.188.16.48"
        DEPLOY_DIR   = "/home/apiadmin/deploy/DEPLOY-APIFORLAB"
        
        // --- แก้ไขโหมดตรงนี้ก่อน Push ---
        // 1 = Customer, 2 = Invoice, 3 = Order, 4 = Payment
        DEPLOY_MODE = "1" 
    }

    stages {
        stage('Determine Service') {
            steps {
                script {
                    // ใช้เงื่อนไขเช็กจากตัวเลขที่เราพิมพ์ไว้ด้านบน
                    if (env.DEPLOY_MODE == "1") {
                        env.SERVICE_DIR = "Customer"
                        env.PROJECT_NAME = "customer"
                    } else if (env.DEPLOY_MODE == "2") {
                        env.SERVICE_DIR = "Invoice"
                        env.PROJECT_NAME = "invoice"
                    } else if (env.DEPLOY_MODE == "3") {
                        env.SERVICE_DIR = "Order"
                        env.PROJECT_NAME = "order"
                    } else if (env.DEPLOY_MODE == "4") {
                        env.SERVICE_DIR = "Payment"
                        env.PROJECT_NAME = "payment"
                    } else {
                        error "❌ เลขโหมดไม่ถูกต้อง! กรุณาใส่ 1-4 เท่านั้น"
                    }
                }
            }
        }

        stage('Sync and Deploy') {
            steps {
                sshagent(['docker-server']) {
                    sh '''
                        set -eux
                        echo "🚀 Mode: ${DEPLOY_MODE} -> Deploying: ${SERVICE_DIR}"

                        # สร้างโฟลเดอร์ที่เครื่องปลายทาง
                        ssh -o StrictHostKeyChecking=no ${DOCKER_HOST} "mkdir -p ${DEPLOY_DIR}/${SERVICE_DIR}"

                        # rsync เฉพาะ Folder ที่เราเลือก
                        rsync -avz --delete -e "ssh -o StrictHostKeyChecking=no" \
                            "${SERVICE_DIR}/" "${DOCKER_HOST}:${DEPLOY_DIR}/${SERVICE_DIR}/"

                        # สั่ง Docker Compose Up
                        ssh -o StrictHostKeyChecking=no ${DOCKER_HOST} "
                            set -eux
                            cd '${DEPLOY_DIR}/${SERVICE_DIR}'
                            if [ -f 'docker-compose.yaml' ]; then
                                docker compose -p '${PROJECT_NAME}' up -d --remove-orphans
                                docker ps --format 'table {{.Names}}\\t{{.Status}}'
                            else
                                echo '❌ ไม่พบไฟล์ docker-compose.yaml!'
                                exit 1
                            fi
                        "
                    '''
                }
            }
        }
    }
}