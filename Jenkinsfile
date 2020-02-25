@Library('defra-library@0.0.13')
import uk.gov.defra.ffc.DefraUtils
def defraUtils = new DefraUtils()

// Versioning - edit these variables to set version information
def dockerfileVersion = '1.0.0'
def nodeVersion = '12.16.0'

// Constants
def awsRegion = 'eu-west-2'
def imageName = 'ffc-node'
def imageNameDevelopment = 'ffc-node-development'
def regCredsId = 'ecr:eu-west-2:ecr-user'
def registry = '562955126301.dkr.ecr.eu-west-2.amazonaws.com'
def repoName = 'ffc-docker-node'

// Variables
def containerTag = ''
def imageTag = ''
def mergedPrNo = ''
def pr = ''

node {
  checkout scm

  try {
    stage('Set variables') {
      imageTag = "$dockerfileVersion-node${nodeVersion}"
      (pr, containerTag, mergedPrNo) = defraUtils.getVariables(repoName, imageTag)
      defraUtils.setGithubStatusPending()

      imageRepository = "$registry/$imageName"
      imageRepositoryDevelopment = "$registry/$imageNameDevelopment"
      imageTag = imageTag + (pr ? "-pr$pr" : "")
    }

    stage('Build') {
      sh "docker build --no-cache --tag $imageRepository:$imageTag --build-arg NODE_VERSION=${nodeVersion} \
      --build-arg VERSION=$dockerfileVersion --target production . "

      sh "docker build --no-cache --tag $imageRepositoryDevelopment:$imageTag --build-arg NODE_VERSION=${nodeVersion} \
      --build-arg VERSION=$dockerfileVersion --target development . "
    }

    stage('Push') {
      docker.withRegistry("https://$registry", regCredsId) {
        sh "docker push $imageRepository:$imageTag"
        sh "docker push $imageRepositoryDevelopment:$imageTag"
      }
    }

    if (mergedPrNo) {
      // Remove PR image tags from registry after merge to master.
      // Leave digests as these will be reused by master build or cleaned up automatically.
      prImageTag = "$dockerfileVersion-node${nodeVersion}-$mergedPrNo"
      stage('Clean registry') {
        sh """
          aws --region $awsRegion \
            ecr batch-delete-image \
            --image-ids imageTag=$prImageTag \
            --repository-name $imageName
        """
        sh """
          aws --region $awsRegion \
            ecr batch-delete-image \
            --image-ids imageTag=$prImageTag \
            --repository-name $imageNameDevelopment
        """
      }
      stage('Tag release in github') {
        withCredentials([
          string(credentialsId: 'github_ffc_platform_repo', variable: 'gitToken')
        ]) {
          defraUtils.triggerRelease(imageTag, repoName, imageTag, gitToken)
        }
      }
    }

    defraUtils.setGithubStatusSuccess()
  } catch(e) {
    defraUtils.setGithubStatusFailure(e.message)
    throw e
  }
}
