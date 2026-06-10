pipeline{     
   agent any

   stages{     

        stage('Checkov') {
            steps {
                sh '''
                    BASE="$HOME/.jenkins-tools"
                    REPO="$BASE/reports"

                    mkdir -p "$REPO/reports"

                    echo "== Installation de Checkov =="
                    pipx install --quiet checkov || true
                    pipx ensurepath
                    export PATH="$PATH:$HOME/.local/bin"

                    echo "== Vérification Checkov =="
                    which checkov || ls -l $HOME/.local/bin

                    echo "== Scan Checkov au format SARIF =="
                    checkov -d . -o sarif \
                        --output-file "result.sarif" \
                        --quiet --soft-fail

                    mv result.sarif/* "$REPO/checkov.sarif"
                '''
            }
        }
        
        stage('Move SARIF to workspace') {
            steps {
                sh '''
                mkdir -p "$WORKSPACE/reports"
                mv "$HOME/.jenkins-tools/reports/"*.sarif "$WORKSPACE/reports/" || true
                '''
            }
        }

        stage('Publish SARIF') {
            steps {
                recordIssues enabledForFailure: true, tools: [sarif(pattern: "reports/*.sarif")]
                sh 'rm -rf reports'
            }
        }

        stage('SonarQube') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'
                        withSonarQubeEnv('SonarQube') {
                            withCredentials([
                                string(credentialsId: 'example_sonar_token', variable:'EXAMPLE_SONAR_TOKEN')]){
                                sh """
                                    echo '=== Construction liste SARIF ==='
                                    SARIFS=\$(ls $HOME/.jenkins-tools/reports/*.sarif 2>/dev/null | paste -sd, - || true)
                                    echo "SARIFS = \$SARIFS"
                                    echo '=== Lancement du scanner SonarQube ==='
                                    "${scannerHome}/bin/sonar-scanner" \
                                        -Dsonar.projectKey=tst-christel \
                                        -Dsonar.sources=. \
                                        -Dsonar.host.url=https://sonarqube-tst.off.stib-mivb.net \
                                        -Dsonar.token=sqp_2cb6262df7e46a6df0da507388249000a6a2bead
                                    """
                                }
                    }
                }
            }
        }
   }
}