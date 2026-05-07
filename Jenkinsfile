// ============================================
// Jenkins CI/CD Pipeline
// Linux Compatible Version
// UI + API Hybrid Automation Framework
// ============================================

pipeline {

    agent any

    parameters {

        choice(
            name: 'BROWSER',
            choices: ['chrome', 'firefox', 'edge'],
            description: 'Browser for UI tests'
        )

        choice(
            name: 'ENV',
            choices: ['dev', 'staging', 'production'],
            description: 'Target environment'
        )

        booleanParam(
            name: 'HEADLESS',
            defaultValue: true,
            description: 'Run browser in headless mode'
        )

        string(
            name: 'PARALLEL_WORKERS',
            defaultValue: '2',
            description: 'Number of parallel workers'
        )
    }

    environment {

        TEST_ENV = "${params.ENV}"
        BROWSER = "${params.BROWSER}"
        HEADLESS = "${params.HEADLESS}"
        PYTHONPATH = "${WORKSPACE}"
    }

    stages {

        // ============================================
        // Clean Workspace
        // ============================================

        stage('Clean Workspace') {

            steps {

                cleanWs()

                echo 'Workspace cleaned successfully'
            }
        }

        // ============================================
        // Checkout Source Code
        // ============================================

        stage('Checkout') {

            steps {

                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/RAKESH-564/Capstone-Project_QA1.git'
                    ]]
                ])

                echo 'Code checked out successfully from GitHub repository'
            }
        }

        // ============================================
        // Setup Python Environment
        // ============================================

        stage('Setup Environment') {

            steps {

                sh '''
                    python3 -m venv venv
                    . venv/bin/activate

                    python --version
                    pip --version

                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''

                echo 'Python environment setup completed'
            }
        }

        // ============================================
        // API Health Check
        // ============================================

        stage('API Health Check') {

            steps {

                sh '''
                    . venv/bin/activate

                    python -c "
import requests
r = requests.get('https://practice.expandtesting.com/notes/api/health-check')
print(f'API Status Code: {r.status_code}')
"
                '''

                echo 'API health check completed'
            }
        }

        // ============================================
        // Run Automation Tests
        // ============================================

        stage('Run Tests') {

            steps {

                sh """
                    . venv/bin/activate

                    pytest tests/ \
                    -v \
                    -s \
                    --tb=long \
                    --x \
                    --junitxml=reports/results.xml \
                    --alluredir=reports/allure-results \
                    --html=reports/report.html \
                    --self-contained-html \
                    -n ${params.PARALLEL_WORKERS} \
                    --dist=loadfile \
                    --reruns=2 \
                    --reruns-delay=2
                """
            }
        }

        // ============================================
        // Generate Allure HTML Report
        // ============================================

        stage('Generate Allure Report') {

            steps {

                sh '''
                    . venv/bin/activate

                    allure generate reports/allure-results \
                    -o reports/allure-report \
                    --clean
                '''

                echo 'Allure report generated successfully'
            }
        }

        // ============================================
        // Publish HTML Reports
        // ============================================

        stage('Publish Reports') {

            steps {

                publishHTML([
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'reports',
                    reportFiles: 'report.html',
                    reportName: 'Pytest HTML Report'
                ])

                publishHTML([
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'reports/allure-report',
                    reportFiles: 'index.html',
                    reportName: 'Allure HTML Report'
                ])
            }
        }
    }

    // ============================================
    // Post Actions
    // ============================================

    post {

        always {

            echo 'Archiving test artifacts...'

            archiveArtifacts(
                artifacts: 'reports/**/*',
                allowEmptyArchive: true
            )

            junit(
                testResults: 'reports/results.xml',
                allowEmptyResults: true
            )
        }

        success {

            echo '✅ All tests PASSED!'
        }

        failure {

            echo '❌ Some tests FAILED. Check reports for details.'
        }

        cleanup {

            cleanWs()
        }
    }
}
