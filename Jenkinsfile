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
   }
}