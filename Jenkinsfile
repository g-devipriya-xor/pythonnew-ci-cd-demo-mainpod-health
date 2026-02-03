pipeline {
    agent any

    environment {
        KUBECONFIG = "/home/devipriya/.kube/config"
        IMAGE_NAME = "python-cicd-app"
        NOTIFY_EMAIL = "G.Devipriya@Xoriant.Com"
        CC_EMAILS = "Ashish.Kamath@Xoriant.Com"
        MAIN_BRANCH = "main"
        LOCAL_PORT = "5005"  
        POD_NAME = ""         
    }

    stages {
        stage('Check Main Pod Health') {
            steps {
                script {
                    echo "Checking main branch pod health..."

                    // Get the main pod name
                    POD_NAME = sh(
                        script: "kubectl get pods -l app=${IMAGE_NAME}-${MAIN_BRANCH} -o jsonpath='{.items[0].metadata.name}'",
                        returnStdout: true
                    ).trim()

                    echo "Main pod: ${POD_NAME}"

                    // Get pod status
                    def mainPodStatus = sh(
                        script: "kubectl get pod ${POD_NAME} -o wide",
                        returnStdout: true
                    ).trim()

                    //Get failing pods (if any)
                    def failingMainPods = sh(
                        script: "kubectl get pods -l app=${IMAGE_NAME}-${MAIN_BRANCH} --field-selector=status.phase!=Running",
                        returnStdout: true
                    ).trim()

                    //Start port-forward in background
                    sh """
                    kubectl port-forward ${POD_NAME} ${LOCAL_PORT}:5001 &
                    PF_PID=\$!
                    sleep 5  
                    """

                    // Query /sendpoint via port-forward
                    def appStatus = sh(
                        script: "curl -s http://localhost:${LOCAL_PORT}/",
                        returnStdout: true
                    ).trim()
		    // Get /status endpoint status
		    def appStatusEndpoint = sh(
			script: "curl -s http://localhost:5005/status",
			returnStdout: true
		    ).trim()

                    //Kill port-forward process
                    sh """
                    kill \$PF_PID || true
                    """

                    // Save report
                    writeFile file: 'main_pod_health_report.txt', text: """
                         Main Pod Status 
                    ${mainPodStatus}

                     Failing Main Pods 
                    ${failingMainPods}

                     Main App / Endpoint 
                    ${appStatus}

		     Main App /status Endpoint
		     ${appStatusEndpoint}
                    """
                }
            }
        }

        stage('Send Email Report') {
            steps {
		script{
			def recipients = "${NOTIFY_EMAIL},${CC_EMAILS}".replaceAll("\\s+", "")
			emailext(
				to: recipients,
				subject: "Daily Main Branch Pod Health Report",
				body: """
				<p>Hello Team,</p>
				<p>Here is the daily health report for the <b>Main Branch</b> deployment:</p>
				<pre>${readFile('main_pod_health_report.txt')}</pre>
				<p>Regards,<br>Devipriya</p>
				""",
				mimeType: 'text/html'
				)
			}
            }
        }
    }
}
