@Library('sharedlibrary')_

pipeline {
    parameters {
        string 'GIT_URL'
        string 'BRANCH_NAME'
        string 'repository'       
    }
    environment {
        gitRepoURL = "${params.GIT_URL}"
        gitBranchName = "${params.BRANCH_NAME}"
        repoName = "${params.repository}"
        dockerImage = "654654138345.dkr.ecr.us-east-1.amazonaws.com/${repoName}"
        gitCommit = "${GIT_COMMIT[0..6]}"
        dockerTag = "${params.BRANCH_NAME}-${gitCommit}"
    }
     

    agent {label 'slave-1'}
    stages {
        stage('Git Checkout') {
            steps {
                gitCheckout("$gitRepoURL", "refs/heads/$gitBranchName", 'githubCred')
            }
        }

        stage('Docker Build') {
            steps {
                    dockerImageBuild('$dockerImage', '$dockerTag')
            }
        }

        stage('Docker Push') {
            steps {
                dockerECRImagePush('$dockerImage', '$dockerTag', '$repoName', 'awsCred', 'us-east-1')
            }
        }

        stage('Kubernetes Deploy - DEV') {
            when {
                branch 'development'
            }
            steps {
                kubernetesEKSHelmDeploy('$dockerImage', '$dockerTag', '$repoName', 'awsCred', 'us-east-1', 'test1', 'dev')
            }
        }

    }
}
