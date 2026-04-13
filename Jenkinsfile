pipeline {
    agent any

    environment {
        DOCKER_IMAGE   = "varuna10/varunmohanbcs34-wine-quality:latest"
        YOUR_NAME      = "Varun Mohan"
        YOUR_ROLL_NO   = "2022BCS0034"
        CONTAINER_NAME = "wine-api-${BUILD_NUMBER}"
    }

    stages {

        stage('Pull Image') {
            steps {
                echo "============================================"
                echo "STAGE 1: Pulling Docker image"
                echo "Name: ${YOUR_NAME} | Roll No: ${YOUR_ROLL_NO}"
                echo "============================================"
                sh "docker pull ${DOCKER_IMAGE}"
                sh "docker images ${DOCKER_IMAGE}"
            }
        }

        stage('Run Container') {
            steps {
                echo "============================================"
                echo "STAGE 2: Starting container"
                echo "Name: ${YOUR_NAME} | Roll No: ${YOUR_ROLL_NO}"
                echo "============================================"
                sh "docker run -d --name ${CONTAINER_NAME} ${DOCKER_IMAGE}"
                sh "docker ps --filter name=${CONTAINER_NAME}"
            }
        }

        stage('Wait for Readiness') {
            steps {
                echo "============================================"
                echo "STAGE 3: Waiting for API to be ready"
                echo "Name: ${YOUR_NAME} | Roll No: ${YOUR_ROLL_NO}"
                echo "============================================"
                sh """
                    CONTAINER_IP=\$(docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' ${CONTAINER_NAME})
                    echo "Container IP: \$CONTAINER_IP"

                    TIMEOUT=30
                    ELAPSED=0
                    until curl -sf http://\$CONTAINER_IP:8000/health; do
                        if [ \$ELAPSED -ge \$TIMEOUT ]; then
                            echo "ERROR: API not ready after \${TIMEOUT}s"
                            exit 1
                        fi
                        echo "Not ready yet... waiting 5s (elapsed: \${ELAPSED}s)"
                        sleep 5
                        ELAPSED=\$((ELAPSED + 5))
                    done
                    echo "--- API is READY ---"
                """
            }
        }

        stage('Valid Inference') {
            steps {
                echo "============================================"
                echo "STAGE 4: Sending valid inference request"
                echo "Name: ${YOUR_NAME} | Roll No: ${YOUR_ROLL_NO}"
                echo "============================================"
                sh """
                    CONTAINER_IP=\$(docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' ${CONTAINER_NAME})
                    echo "Container IP: \$CONTAINER_IP"

                    RESPONSE=\$(curl -s -w "\\n%{http_code}" -X POST "http://\$CONTAINER_IP:8000/predict" \\
                        -H "Content-Type: application/json" \\
                        -d '{
                            "fixed_acidity": 7.4,
                            "volatile_acidity": 0.70,
                            "citric_acid": 0.00,
                            "residual_sugar": 1.9,
                            "chlorides": 0.076,
                            "free_sulfur_dioxide": 11.0,
                            "total_sulfur_dioxide": 34.0,
                            "density": 0.9978,
                            "pH": 3.51,
                            "sulphates": 0.56,
                            "alcohol": 9.4
                        }')

                    HTTP_BODY=\$(echo "\$RESPONSE" | head -n -1)
                    HTTP_CODE=\$(echo "\$RESPONSE" | tail -n 1)

                    echo "Raw Response: \$HTTP_BODY"
                    echo "HTTP Status: \$HTTP_CODE"

                    if [ "\$HTTP_CODE" != "200" ]; then
                        echo "FAIL: Expected HTTP 200 but got \$HTTP_CODE"
                        exit 1
                    fi

                    WINE_QUALITY=\$(echo "\$HTTP_BODY" | python3 -c "
import sys, json
data = json.load(sys.stdin)
val = data.get('wine_quality') or data.get('prediction') or data.get('quality')
print(val if val is not None else 'NOT_FOUND')
")

                    if [ "\$WINE_QUALITY" = "NOT_FOUND" ]; then
                        echo "FAIL: wine_quality not found in response"
                        exit 1
                    fi

                    echo ""
                    echo "============================================"
                    echo "INFERENCE RESULT:"
                    echo "{ name: '${YOUR_NAME}', roll_no: '${YOUR_ROLL_NO}', wine_quality: \$WINE_QUALITY }"
                    echo "============================================"
                """
            }
        }

        stage('Invalid Input Test') {
            steps {
                echo "============================================"
                echo "STAGE 5: Sending INVALID request"
                echo "Name: ${YOUR_NAME} | Roll No: ${YOUR_ROLL_NO}"
                echo "Expecting 4xx or 5xx error"
                echo "============================================"
                sh """
                    CONTAINER_IP=\$(docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' ${CONTAINER_NAME})

                    RESPONSE=\$(curl -s -w "\\n%{http_code}" -X POST "http://\$CONTAINER_IP:8000/predict" \\
                        -H "Content-Type: application/json" \\
                        -d '{"bad_field": "bad_value"}')

                    HTTP_BODY=\$(echo "\$RESPONSE" | head -n -1)
                    HTTP_CODE=\$(echo "\$RESPONSE" | tail -n 1)

                    echo "Raw Response: \$HTTP_BODY"
                    echo "HTTP Status: \$HTTP_CODE"

                    if echo "\$HTTP_CODE" | grep -qE "^[45][0-9][0-9]\$"; then
                        echo "PASS: API correctly returned error \$HTTP_CODE for invalid input"
                    else
                        echo "FAIL: Expected 4xx/5xx but got \$HTTP_CODE"
                        exit 1
                    fi
                """
            }
        }

        stage('Stop Container') {
            steps {
                echo "============================================"
                echo "STAGE 6: Stopping and removing container"
                echo "Name: ${YOUR_NAME} | Roll No: ${YOUR_ROLL_NO}"
                echo "============================================"
                sh """
                    docker stop ${CONTAINER_NAME}
                    docker rm ${CONTAINER_NAME}
                    LEFTOVER=\$(docker ps --filter name=${CONTAINER_NAME} --format "{{.Names}}")
                    if [ -z "\$LEFTOVER" ]; then
                        echo "VERIFIED: Container removed. Clean!"
                    else
                        echo "FAIL: Container still running!"
                        exit 1
                    fi
                """
            }
        }

    }

    post {
        success {
            echo "============================================"
            echo "PIPELINE STATUS: SUCCESS"
            echo "All 6 stages passed!"
            echo "Name: ${YOUR_NAME} | Roll No: ${YOUR_ROLL_NO}"
            echo "============================================"
        }
        failure {
            echo "============================================"
            echo "PIPELINE STATUS: FAILURE"
            echo "Name: ${YOUR_NAME} | Roll No: ${YOUR_ROLL_NO}"
            echo "============================================"
            sh "docker stop ${CONTAINER_NAME} || true"
            sh "docker rm ${CONTAINER_NAME} || true"
        }
        always {
            echo "Pipeline finished. Name: ${YOUR_NAME} | Roll No: ${YOUR_ROLL_NO}"
        }
    }
}