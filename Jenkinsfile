pipeline {
    agent any
    
    stages {
        stage('Verificar Docker y Docker Compose') {
            steps {
                sh '''
                docker --version
                docker-compose --version
                '''
            }
        }

        stage('Clonar el repositorio') {
            steps {
                git branch: 'main', url: 'https://github.com/anaCami13/APIREST.git'
            }
        }

        stage('Instalar Docker Compose') {
            steps {
                sh '''
                curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
                chmod +x /usr/local/bin/docker-compose
                '''
            }
        }

        stage('Construir y Levantar Servicios con Docker Compose') {
            steps {
                script {
                    // Construir y levantar los servicios definidos en docker-compose.yml
                    sh 'docker-compose down' // Baja cualquier contenedor previo
                    sh 'docker-compose build'
                    sh 'docker-compose up -d' // Levanta y construye los contenedores
                }
                sleep(time: 30, unit: 'SECONDS') // Pausa para permitir que los servicios se inicien
            }
        }

        stage('Verificar si los contenedores están corriendo') {
            steps {
                sh 'docker ps' // Verifica que los contenedores estén corriendo
            }
        }

        stage('Esperar a que el servicio esté disponible') {
            steps {
                script {
                    def maxRetries = 10
                    def retryCount = 0
                    def success = false
                    while (retryCount < maxRetries) {
                        try {
                            // Intenta hacer una conexión con la API
                            sh 'curl -f http://10.0.30.102:3000/tasks'
                            success = true
                            break
                        } catch (Exception e) {
                            echo "Esperando que el servicio esté disponible..."
                            retryCount++
                            sleep(time: 30, unit: 'SECONDS')  // Espera antes de volver a intentar
                        }
                    }
                    if (!success) {
                        error "El servicio no está disponible después de ${maxRetries} intentos."
                    }
                }
            }
        }

        stage('Ejecutar Pruebas') {
            steps {
                script {
                    // Pruebas de tareas
                    sh 'curl -f http://10.0.30.102:3000/tasks'
                    sh '''
                        curl -X POST http://10.0.30.102:3000/tasks \
                        -H "Content-Type: application/json" \
                        -d '{
                            "Name": "Nueva Tarea",
                            "Content": "Contenido de la tarea"
                        }'
                    '''
                    sh 'curl -f http://10.0.30.102:3000/tasks/1'
                    sh '''
                        curl -X PUT http://10.0.30.102:3000/tasks/1 \
                        -H "Content-Type: application/json" \
                        -d '{
                            "Name": "Tarea Actualizada",
                            "Content": "Contenido actualizado"
                        }'
                    '''
                    sh 'curl -X DELETE http://10.0.30.102:3000/tasks/1'
                }
            }
        }

        stage('Detener Servicios') {
            steps {
                script {
                    sh 'docker-compose down' // Detiene los contenedores después de las pruebas
                }
            }
        }
    }

    post {
        success {
            echo 'Las pruebas han sido exitosas.'
        }
        failure {
            echo 'Hubo errores en las pruebas.'
        }
    }
}
