pipeline {
    agent any

    environment {
        KUBECONFIG = "/home/devipriya/.kube/config"
        IMAGE_NAME = "python-cicd-app"
        NOTIFY_EMAIL = "G.Devipriya@Xoriant.Com"
        CC_EMAILS = "Ashish.Kamath@Xoriant.Com"
        MAIN_BRANCH = "main"
    }

    stages {
        stage('Check Main Pod Health') {
            steps {
                script {
                    echo "Checking main branch pod health..."

                    // Get pods info: Name, Status, Node, IP
                    def podInfo = sh(
                        script: """
                        kubectl get pods -l app=${IMAGE_NAME}-${MAIN_BRANCH} \\
                        -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName,IP:.status.podIP --no-headers
                        """,
                        returnStdout: true
                    ).trim()

                    // Build HTML table
                    def tableHtml = """
                    <table border="1" cellpadding="5" cellspacing="0">
                        <thead>
                            <tr>
                                <th>Pod Name</th>
                                <th>Status</th>
                                <th>Node</th>
                                <th>Pod IP</th>
                            </tr>
                        </thead>
                        <tbody>
                    """

                    podInfo.split("\n").each { line ->
                        def cols = line.split("\\s+")
                        tableHtml += "<tr><td>${cols[0]}</td><td>${cols[1]}</td><td>${cols[2]}</td><td>${cols[3]}</td></tr>"
                    }

                    tableHtml += "</tbody></table>"

                    // Save report
                    writeFile file: 'main_pod_health_report.html', text: tableHtml
                }
            }
        }

        stage('Send Email Report') {
            steps {
                script {
                    def recipients = "${NOTIFY_EMAIL},${CC_EMAILS}".replaceAll("\\s+", "")
                    def reportContent = readFile('main_pod_health_report.html')
                    emailext(
                        to: recipients,
                        subject: "Daily Main Branch Pod Health Report",
                        body: """
                        <p>Hello Team,</p>
                        <p>Here is the daily health report for the <b>Main Branch</b> deployment:</p>
                        ${reportContent}
                        <p>Regards,<br>Devipriya</p>
                        """,
                        mimeType: 'text/html'
                    )
                }
            }
        }
    }
}
