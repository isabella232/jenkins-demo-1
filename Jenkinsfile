pipeline {
    agent any
    stages {
        stage('terraform') {
            agent {
                docker {
                    image 'hashicorp/terraform:latest'
                    args "--user=root --entrypoint=''"
                }
            }

            // IMPORTANT: add any required cloud credentials
            environment {
                AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
                AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
            }

            steps {
                sh 'cdterraform'
                sh 'terraform init'
                sh 'terraform plan -out=tfplan.binary'
                sh 'terraform show -json tfplan.binary > plan.json'
                stash includes: 'plan.json', name: 'plan_json'
            }
        }

        stage('infracost') {
            agent {
                docker {
                    // Always use the latest 0.9.x version to pick up bug fixes and new resources.
                    // See https://www.infracost.io/docs/integrations/cicd/#docker-images for other options
                    image 'infracost/infracost:ci-0.9'
                    args "--user=root --entrypoint=''"
                }
            }

            // Set up any required credentials for posting the comment, e.g. GitHub token, GitLab token
            environment {
                INFRACOST_API_KEY = credentials('jenkins-infracost-api-key')
            }

            steps {
                unstash 'plan_json'

                // Generate Infracost JSON output, the following docs might be useful:
                // Multi-project/workspaces: https://www.infracost.io/docs/features/config_file
                // Combine Infracost JSON files: https://www.infracost.io/docs/features/cli_commands/#combined-output-formats
                sh 'infracost breakdown --path plan.json --format json --out-file infracost.json'

                // IMPORTANT: update this depending on which VCS provider you use and which plugin you are using.
                // Infracost comment supports GitHub, GitLab, Azure Repos and Bitbucket
                // You will need to update the environment variables below to match your environment.
                // For the full list of options, see: https://www.infracost.io/docs/features/cli_commands/#comment-on-pull-requests
                // sh 'infracost comment github --path infracost.json --repo $GITHUB_REPO --pull-request $GITHUB_PULL_REQUEST_NUMBER --github-token $GITHUB_TOKEN'
            }
        }
    }
}
