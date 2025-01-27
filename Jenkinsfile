pipeline {
    agent any

    environment {
        CONTAINER_ID = '' // Variable pour stocker l’ID du conteneur
        SUM_PY_PATH = './sum.py' // Chemin local vers sum.py
        DIR_PATH = './' // Chemin local vers le Dockerfile
        TEST_FILE_PATH = './test_variables.txt' // Chemin local vers le fichier de test
    }

    stages {
        stage('Build') {
            steps {
                script {
                    // Construction de l’image Docker
                    sh "docker build -t sum-python ${DIR_PATH}"
                }
            }
        }

        stage('Run') {
            steps {
                script {
                    // Exécution du conteneur et stockage de l’ID
                    def output = sh(script: "docker run -dit sum-python", returnStdout: true).trim()
                    CONTAINER_ID = output
                    echo "Conteneur démarré avec ID : ${CONTAINER_ID}"
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    // Lecture du fichier de test
                    def testLines = readFile(TEST_FILE_PATH).split('\n')
                    for (line in testLines) {
                        if (line.trim()) { // Ignorer les lignes vides
                            def vars = line.split(' ')
                            def arg1 = vars[0]
                            def arg2 = vars[1]
                            def expectedSum = vars[2].toFloat()

                            // Exécution du script Python à l’intérieur du conteneur
                            def output = sh(
                                script: "docker exec ${CONTAINER_ID} python /app/sum.py ${arg1} ${arg2}",
                                returnStdout: true
                            ).trim()
                            def result = output.toFloat()

                            // Comparaison avec la somme attendue
                            if (result == expectedSum) {
                                echo "Test réussi : ${arg1} + ${arg2} = ${expectedSum}"
                            } else {
                                error "Test échoué : ${arg1} + ${arg2} = ${expectedSum}, mais obtenu ${result}"
                            }
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                // Arrêter et supprimer le conteneur après le pipeline
                sh "docker stop ${CONTAINER_ID}"
                sh "docker rm ${CONTAINER_ID}"
            }
        }
    }
}
