pipeline {
    agent any

    environment {
        // Nombre interno de la imagen en Minikube (no requiere usuario externo)
        DOCKER_IMAGE = 'node-api-rest'
        
        // OPCIÓN RECOMENDADA PARA ENTORNO LOCAL (MINIKUBE):
        // Usa una etiqueta estática (ej. el nombre de la rama) para no llenar el disco con múltiples imágenes.
        // Nota: requiere usar 'rollout restart' más abajo para forzar la actualización en Kubernetes.
        BRANCH_TAG = "${env.BRANCH_NAME.replace('/', '-')}"
        
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test & SCA') {
          agent {
            docker {
                image 'node:20-alpine'
                args '-u root:root --entrypoint=""' // Ejecutar como root para evitar problemas de permisos con node_modules
                reusdeNode true // Reutilizar el contenedor para mantener node_modules entre etapas
            }
          }
            steps {
              script {
                echo "Instalando dependencias y ejecutando tests dentro del contenedor Docker..."
                sh 'npm ci'
                sh 'npm run test:ci'
                
                echo "Verificando vulnerabilidades con npm audit..."
                sh 'npm audit --audit-level=high' // Falla si se encuentran vulnerabilidades de nivel alto o crítico
              }
            }
        }

        stage('SAST - SonarQube') {
            steps {
              script {
                echo "Analizando el código fuente con SonarQube (SAST)..."
                def scannerHome = tool 'sonar-scanner'
                withSonarQubeEnv('sonarqube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
              }
            }
        }

        stage('Construir y Cargar Imagen en Minikube') {
            steps {
                script {
                    echo "Construyendo la imagen localmente usando Docker del host..."
                    sh "docker build -t ${DOCKER_IMAGE}:${BRANCH_TAG} ."
                    
                    echo "Cargando la imagen al daemon Docker de Minikube..."
                    // Cargamos la imagen al daemon de Docker de Minikube usando docker exec.
                    // Esto soluciona el issue de permisos donde el usuario de jenkins no tiene 
                    // acceso al perfil de minikube del usuario host, pero sí al servicio docker.
                    sh "docker save ${DOCKER_IMAGE}:${BRANCH_TAG} | docker exec -i minikube docker load"
                }
            }
        }

        stage('Container Scanning - Trivy') {
            steps {
                script {
                    echo "Escaneando la imagen Docker en busca de CVES (Container Scanning)..."
                    // Ejecutamos Trivy directamente contra la imagen cargada en Minikube
                    sh """
                        docker run --rm \
                            -v /var/run/docker.sock:/var/run/docker.sock \
                            aquasec/trivy:0.69.3 image \
                            --exit-code 1 \
                            --severity CRITICAL \
                            --no-progress \
                            ${DOCKER_IMAGE}:${BRANCH_TAG}
                    """
                }
            }
        }
        // Se elimina la fase de Push a Docker Hub porque la imagen ya vive dentro del clúster
        
        stage('Desplegar en K8s (Minikube)') {
            steps {
                // Usamos el plugin Kubernetes CLI
                withKubeConfig([credentialsId: 'k8s-token', serverUrl: 'https://192.168.49.2:8443']) {
                    script {
                        def namespace = 'default'
                        if (env.BRANCH_NAME == 'main') {
                            namespace = 'prod'
                        } else if (env.BRANCH_NAME == 'develop') {
                            namespace = 'dev'
                        }

                        // Asegurarnos que el Namespace exista
                        sh "kubectl create namespace ${namespace} --dry-run=client -o yaml | kubectl apply -f -"
                        
                        // Actualizar la imagen en el Deployment
                        sh "sed -i 's|image: .*|image: ${DOCKER_IMAGE}:${BRANCH_TAG}|g' k8s/api-deployment.yaml"
                        
                        // Aplicar los cambios
                        sh "kubectl apply -f k8s/ -n ${namespace}"
                        
                        // TRUCO PARA MANTENER LA MISMA ETIQUETA LOCALMENTE:
                        // Como estamos reemplazando la imagen pero la etiqueta es dinámica (misma rama, mismo texto), 
                        // K8s no detecta "cambios". Forzamos el reinicio de los Pods para que asimilen la nueva build.
                        // *Nota: Asegúrate de reemplazar 'node-api' con el nombre de tu deployment.
                        sh "kubectl rollout restart deployment node-api -n ${namespace}"
                    }
                }
            }
        }
    }
}