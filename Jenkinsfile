// =============================================================================
//  Pipeline Jenkins - BACKEND (Spring Boot / Maven / Java 17)
//  Dépôt : DevOps-AppGestionDesProjets   |   Module : backend/
//
//  1. Déclenchement automatique à chaque push GitHub (webhook)
//  2. Récupération du projet depuis GitHub
//  3. Exécution des tests unitaires
//  4. Génération du livrable (JAR) dans backend/target
// =============================================================================
pipeline {
    agent any

    // --- Outils déclarés dans : Manage Jenkins > Tools -----------------------
    //     (adapter les noms 'Maven3' et 'JDK17' à votre configuration)
    tools {
        maven 'M3'
        jdk   'JDK21'
    }

    // --- 1) Détection du push GitHub -----------------------------------------
    //     Nécessite le plugin "GitHub" + un webhook GitHub pointant vers
    //     http://<URL_JENKINS>/github-webhook/
    triggers {
        githubPush()
    }

    options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        // TODO : remplacer par l'URL de VOTRE dépôt GitHub
        REPO_URL   = 'https://github.com/ahmedhizi/4IET-I2-Ahmed-Hizi.git'
        BRANCH     = 'main'
        BACKEND_DIR = 'backend'
    }

    stages {

        // --- 2) Récupération du projet depuis GitHub -------------------------
        stage('Checkout') {
            steps {
                git branch: "${BRANCH}", url: "${REPO_URL}"
                // Dépôt privé : ajouter  credentialsId: 'github-credentials'
            }
        }

        // --- 3) Tests unitaires ----------------------------------------------
        stage('Tests unitaires') {
            steps {
                dir("${BACKEND_DIR}") {
                    sh 'mvn -B clean test'
                }
            }
            post {
                always {
                    // Publie le rapport de tests dans Jenkins
                    junit testResults: "${BACKEND_DIR}/target/surefire-reports/*.xml",
                          allowEmptyResults: true
                }
            }
        }

        // --- 4) Création du livrable dans le dossier target ------------------
        stage('Package') {
            steps {
                dir("${BACKEND_DIR}") {
                    // Les tests ont déjà été exécutés à l'étape précédente
                    sh 'mvn -B package -DskipTests'
                }
            }
        }

        stage('Archivage du livrable') {
            steps {
                archiveArtifacts artifacts: "${BACKEND_DIR}/target/*.jar",
                                 excludes: "${BACKEND_DIR}/target/*.original",
                                 fingerprint: true
            }
        }
    }

    post {
        success {
            echo "Build OK - livrable : ${BACKEND_DIR}/target/backend-1.0.0.jar"
        }
        failure {
            echo 'Build en échec - consulter la Console Output et le rapport de tests.'
        }
    }
}
