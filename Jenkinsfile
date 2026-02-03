pipeline {
    agent any

    environment {
        KUBECONFIG = "/home/devipriya/.kube/config"
        IMAGE_NAME = "python-cicd-app"
        NOTIFY_EMAIL = "G.Devipriya@Xoriant.Com"
        #CC_EMAILS = "team1@company.com,team2@company.com"
        MAIN_BRANCH = "main"
        LOCAL_PORT = "5005"  // local port for port-forward
        POD_NAME = ""         // will be set dynamically
    }

    triggers {
        cron('23 11 * * *') // daily at 9:00 AM
    }

    stages {
        stage('Check Main Pod Health') {
            steps {
                script {
                    echo "Checking main branch pod health..."

                    // 1️⃣ Get the main pod name
                    POD_NAME = sh(
                        script: "kubectl get pods -l app=${IMAGE_NAME}-${MAIN_BRANCH} -o jsonpath='{.items[0].metadata.name}'",
                        returnStdout: true
                    ).trim()

                    echo "Main pod: ${POD_NAME}"

                    // 2️⃣ Get pod status
                    def mainPodStatus = sh(
                        script: "kubectl get pod ${POD_NAME} -o wide",
                        returnStdout: true
                    ).trim()

                    // 3️⃣ Get failing pods (if any)
                    def failingMainPods = sh(
                        script: "kubectl get pods -l app=${IMAGE_NAME}-${MAIN_BRANCH} --field-selector=status.phase!=Running",
                        returnStdout: true
                    ).trim()

                    // 4️⃣ Start port-forward in background
                    sh """
                    kubectl port-forward ${POD_NAME} ${LOCAL_PORT}:5001 &
                    PF_PID=\$!
                    sleep 5  # give it some time to start
                    """

                    // 5️⃣ Query /status endpoint via port-forward
                    def appStatus = sh(
                        script: "curl -s http://localhost:${LOCAL_PORT}/status",
                        returnStdout: true
                    ).trim()

                    // 6️⃣ Kill port-forward process
                    sh """
                    kill \$PF_PID || true
                    """

                    // 7️⃣ Save report
                    writeFile file: 'main_pod_health_report.txt', text: """
                    ===== Main Pod Status =====
                    ${mainPodStatus}

                    ===== Failing Main Pods =====
                    ${failingMainPods}

                    ===== Main App /status Endpoint =====
                    ${appStatus}
                    """
                }
            }
        }

        stage('Send Email Report') {
            steps {
                emailext(
                    to: "${NOTIFY_EMAIL}",
                    #cc: "${CC_EMAILS}",
                    subject: "Daily Main Branch Pod Health Report",
                    body: """
                    <p>Hello Team,</p>
                    <p>Here is the daily health report for the <b>Main Branch</b> deployment:</p>
                    <pre>${readFile('main_pod_health_report.txt')}</pre>
                    <p>Regards,<br>Jenkins</p>
                    """,
                    mimeType: 'text/html'
                )
            }
        }
    }
}
