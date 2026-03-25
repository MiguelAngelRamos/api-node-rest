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