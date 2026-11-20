pipeline {
    agent any

    tools {
        jdk 'JDK17'
        maven 'Maven'
    }

    environment {
        ARTIFACTORY_URL = 'http://localhost:8082/artifactory'
        ARTIFACTORY_REPO = 'myconstruction-eva3-snapshot'
    }

    stages {

        stage('Verificar entorno JVM') {
            steps {
                bat 'java -version'
                bat 'javac -version'
                bat 'mvn -version'
            }
        }

        stage('Compilar proyecto') {
            steps {
                bat 'mvn -B clean compile'
            }
        }

        stage('Ejecutar pruebas') {
            steps {
                bat 'mvn -B test'
            }
        }

        stage('Generar archivo WAR') {
            steps {
                bat 'mvn -B package -DskipTests'
            }
        }

        stage('Archivar WAR') {
            steps {
                archiveArtifacts(
                    artifacts: 'target/*.war',
                    fingerprint: true,
                    allowEmptyArchive: false
                )
            }
        }

        stage('Publicar WAR en Artifactory') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'artifactory-token',
                        variable: 'ARTIFACTORY_TOKEN'
                    )
                ]) {
                    powershell '''
                        $ErrorActionPreference = 'Stop'

                        $war = Get-Item 'target/MyConstruction-EVA3.war'

                        $destino = "$env:ARTIFACTORY_URL/" +
                            "$env:ARTIFACTORY_REPO/" +
                            "cl/myconstruction/MyConstruction-EVA3/" +
                            "1.0-SNAPSHOT/" +
                            $war.Name

                        $headers = @{
                            Authorization = "Bearer $env:ARTIFACTORY_TOKEN"
                        }

                        Invoke-RestMethod -Uri $destino -Method Put -Headers $headers -InFile $war.FullName -ContentType 'application/octet-stream'

                        Write-Host 'WAR publicado correctamente en Artifactory.'
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'MyConstruction-EVA3 compilado y publicado correctamente.'
        }

        failure {
            echo 'Error durante el pipeline de MyConstruction-EVA3.'
        }
    }
}