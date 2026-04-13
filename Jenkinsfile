pipeline {
    agent any

    environment {
        IMAGE_NAME = "varuna10/varunmohanbcs34-wine-quality:latest"
        CONTAINER_NAME = "wine_api_${BUILD_NUMBER}"
        PORT = "8000"
    }

    stages {

        stage('Pull Image') {
            steps {
                sh 'docker pull $IMAGE_NAME'
            }
        }

        stage('Run Container') {
            steps {
                sh 'docker run -d -p 8000:8000 --name $CONTAINER_NAME $IMAGE_NAME'
            }
        }

        stage('Wait for Readiness') {
            steps {
                sh '''
                for i in {1..6}
                do
                    sleep 5
                    curl -f http://localhost:8000/health && exit 0
                done
                echo "App not ready"
                exit 1
                '''
            }
        }

        stage('Valid Inference') {
            steps {
                sh '''
                RESPONSE=$(curl -s -X POST "http://localhost:8000/predict" \
                -H "Content-Type: application/json" \
                -d '{"fixed_acidity":7.4,"volatile_acidity":0.7,"citric_acid":0,"residual_sugar":1.9,"chlorides":0.076,"free_sulfur_dioxide":11,"total_sulfur_dioxide":34,"density":0.9978,"pH":3.51,"sulphates":0.56,"alcohol":9.4}')

                echo "Raw Response: $RESPONSE"

                VALUE=$(echo $RESPONSE | jq '.wine_quality')

                if [ "$VALUE" = "null" ]; then
                    echo "Error: wine_quality not found"
                    exit 1
                fi

                echo $RESPONSE
                '''
            }
        }

        stage('Invalid Input Test') {
            steps {
                sh '''
                STATUS=$(curl -s -o /dev/null -w "%{http_code}" -X POST "http://localhost:8000/predict" \
                -H "Content-Type: application/json" \
                -d '{}')

                echo "Status Code: $STATUS"

                if [ $STATUS -ge 400 ]; then
                    echo "Correctly failed"
                    exit 1
                else
                    echo "Should have failed"
                    exit 1
                fi
                '''
            }
        }

        stage('Stop Container') {
            steps {
                sh '''
                docker stop $CONTAINER_NAME || true
                docker rm $CONTAINER_NAME || true
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline SUCCESS"
        }
        failure {
            echo "Pipeline FAILED"
        }
    }
}