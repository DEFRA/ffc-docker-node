@Library('defra-library@0.0.8')
import uk.gov.defra.ffc.DefraUtils
def defraUtils = new DefraUtils()

def awsRegion = 'eu-west-2'
def containerTag = ''
def devImageName = 'ffc-node-development'
def imageName = 'ffc-node'
def imageTag = ''
def mergedPrNo = ''
def nodeVersions = ['12.16.0']
def pr = ''
def regCredsId = 'ecr:eu-west-2:ecr-user'
def registry = '562955126301.dkr.ecr.eu-west-2.amazonaws.com'
def repoName = 'ffc-docker-parent'
def version = '1.0.0'

node {
  checkout scm

  try {
    stage('Initialise variables') {
      (pr, containerTag, mergedPrNo) = defraUtils.getVariables(repoName)
      defraUtils.setGithubStatusPending()

      imageName = "ffc-node"
      imageNameDevelopment = "ffc-node-development"
      imageRepository = "$registry/$imageName"
      imageRepositoryDevelopment = "$registry/$devImageName"
      imageTag = "$version-node${nodeVersions[0]}" + (pr ? "-pr$pr" : "")
    }

    stage('Build images') {
      // Build the parent image. 1 parent image per node version required.
      // Tag the images with the PR build (if it's a PR build)
      // Name the image according to the node version used
      // Tag the image according to our version, if it's PR build, include the PR number in the version such as @1.0.0-pr1

      sh "docker build --no-cache --tag $imageRepository:$imageTag --build-arg NODE_VERSION=${nodeVersions[0]} \
      --build-arg VERSION=$version --target production ffc-node-parent/. "

      // Then build the dev images, 1 per node version that reference the parent image that has that node version
      // Name those according to the node version used
      // Tag them with our version
      sh "docker build --no-cache --tag $imageRepositoryDevelopment:$imageTag --build-arg NODE_VERSION=${nodeVersions[0]} \
      --build-arg VERSION=$version --target development ffc-node-parent/. "
    }

    stage('Push images') {
      // Push the parent images
      // Push the development images
      docker.withRegistry("https://$registry", regCredsId) {
        sh "docker push $imageRepository:$imageTag"
        sh "docker push $imageRepositoryDevelopment:$imageTag"
      }
    }

    if (!mergedPrNo) {
      stage('Pretend PR has merged to test ECR image deletion') {
        mergedPrNo="pr$pr"
      }
    }

    if (mergedPrNo) {
      // If this is a merge to master, delete the PR images
      stage('Delete merged PR images') {
        prImageTag = "$version-node${nodeVersions[0]}-$mergedPrNo"
        sh "aws --region $awsRegion ecr batch-delete-image --repository-name $imageRepository --image-ids imageTag=$prImageTag"
        sh "aws --region $awsRegion ecr batch-delete-image --repository-name $imageRepositoryDevelopment --image-ids imageTag=$prImageTag"
      }
    }

    defraUtils.setGithubStatusSuccess()
  } catch(e) {
    defraUtils.setGithubStatusFailure(e.message)
    throw e
  }
}
