@Library('defra-library@3.0.0')
import uk.gov.defra.ffc.DefraUtils
def defraUtils = new DefraUtils()

// Versioning - edit these variables to set version information
def dockerfileVersion = '1.0.0'
def nodeVersion = '12.16.0'

// Constants
def awsCredential = 'devffc-user'
def awsRegion = DEFAULT_AWS_REGION
def imageNameDevelopment = 'ffc-node-development'
def imageNameProduction = 'ffc-node'
def regCredsId = DOCKER_REGISTRY_CREDENTIALS_ID
def registry = DOCKER_REGISTRY
def repoName = 'ffc-docker-node'

// Variables
def imageTag = ''
def imageRepositoryDevelopment = ''
def imageRepositoryProduction = ''
def mergedPrImageTag = ''
def pr = ''
def versionTag = ''

node {
  checkout scm

  try {
    stage('Set variables') {
      versionTag = "${dockerfileVersion}-node${nodeVersion}"
      (pr, imageTag, mergedPrImageTag) = defraUtils.getVariables(repoName, versionTag)
      defraUtils.setGithubStatusPending()

      imageRepositoryDevelopment = "$registry/$imageNameDevelopment"
      imageRepositoryProduction = "$registry/$imageNameProduction"
    }

    stage('Build') {
      sh "docker build --no-cache \
        --tag $imageRepositoryDevelopment:$imageTag \
        --build-arg NODE_VERSION=${nodeVersion} \
        --build-arg VERSION=$dockerfileVersion \
        --target development \
        ."

      sh "docker build --no-cache \
        --build-arg NODE_VERSION=${nodeVersion} \
        --build-arg VERSION=$dockerfileVersion \
        --tag $imageRepositoryProduction:$imageTag \
        --target production \
        ."
    }

    stage('Push') {
      docker.withRegistry("https://$registry", regCredsId) {
        sh "docker push $imageRepositoryDevelopment:$imageTag"
        sh "docker push $imageRepositoryProduction:$imageTag"
      }
    }

    if (mergedPrImageTag) {
      // Remove PR image tags from registry after merge to master.
      // Leave digests as these will be reused by master build or cleaned up automatically.
      prImageTag = "$dockerfileVersion-node${nodeVersion}-$mergedPrImageTag"
      stage('Clean registry') {
        withAWS(credentials: awsCredential, region: awsRegion) {
          sh """
            aws ecr batch-delete-image \
              --image-ids imageTag=$prImageTag \
              --repository-name $imageNameDevelopment
          """
          sh """
            aws ecr batch-delete-image \
              --image-ids imageTag=$prImageTag \
              --repository-name $imageNameProduction
          """
        }
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
