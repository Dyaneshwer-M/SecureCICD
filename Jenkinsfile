// vars/securityScan.groovy

def runGitleaks() {
    def gitleaksImage = 'zricethezav/gitleaks:latest'
    
    sh """
        docker pull ${gitleaksImage}
        docker run --rm -v \${PWD}:/path \
            ${gitleaksImage} detect \
            --source="/path" \
            --report-path="/path/gitleaks-report.json" \
            --report-format=json
    """
    
    // Check if any leaks were found
    def reportFile = readFile('gitleaks-report.json')
    if (reportFile.trim() != '[]') {
        error("Gitleaks found potential secrets in the code!")
    }
}

def runTrivy() {
    def trivyImage = 'aquasec/trivy:latest'
    
    sh """
        docker pull ${trivyImage}
        docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
            -v \${PWD}:/root/scan \
            ${trivyImage} image \
            --format template \
            --template "@/root/scan/html.tpl" \
            -o /root/scan/trivy-report.html \
            ${DOCKER_IMAGE}:${DOCKER_TAG}
    """
    
    // Optional: Fail build on high severity vulnerabilities
    sh """
        docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
            ${trivyImage} image \
            --exit-code 1 \
            --severity HIGH,CRITICAL \
            ${DOCKER_IMAGE}:${DOCKER_TAG}
    """
}