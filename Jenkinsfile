@Library('defra-library@0.0.8')
import uk.gov.defra.ffc.DefraUtils
def defraUtils = new DefraUtils()

def awsRegion = 'eu-west-2'
def containerTag = ''
def imageName = 'ffc-node'
def imageNameDevelopment = 'ffc-node-development'
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
    stage('Set variables') {
      (pr, containerTag, mergedPrNo) = defraUtils.getVariables(repoName)
      defraUtils.setGithubStatusPending()

      imageName = "ffc-node"
      imageNameDevelopment = "ffc-node-development"
      imageRepository = "$registry/$imageName"
      imageRepositoryDevelopment = "$registry/$imageNameDevelopment"
      imageTag = "$version-node${nodeVersions[0]}" + (pr ? "-pr$pr" : "")
    }

    stage('Build') {
      sh "docker build --no-cache --tag $imageRepository:$imageTag --build-arg NODE_VERSION=${nodeVersions[0]} \
      --build-arg VERSION=$version --target production ffc-node-parent/. "

      sh "docker build --no-cache --tag $imageRepositoryDevelopment:$imageTag --build-arg NODE_VERSION=${nodeVersions[0]} \
      --build-arg VERSION=$version --target development ffc-node-parent/. "
    }

    stage('Push') {
      docker.withRegistry("https://$registry", regCredsId) {
        sh "docker push $imageRepository:$imageTag"
        sh "docker push $imageRepositoryDevelopment:$imageTag"
      }
    }

    // Fake PR merge
    if (!mergedPrNo) {
      stage('Fake merge') {
        mergedPrNo="pr$pr"
      }
    }

    if (mergedPrNo) {
      // Remove PR images from registry after merge to master
      stage('Clean registry') {
        prImageTag = "$version-node${nodeVersions[0]}-$mergedPrNo"
        prImageDigest = sh(returnStdout: true, script: """
          aws --region $awsRegion ecr describe-images --image-ids imageTag=$prImageTag --query 'imageDetails[].imageDigest' --repository-name=$imageName
        """)
        prImageDigestDevelopment = sh(returnStdout: true, script: """
          aws --region $awsRegion ecr describe-images --image-ids imageTag=$prImageTag --query 'imageDetails[].imageDigest' --repository-name=$imageNameDevelopment
        """)

        // Delete merged PR tags from images in registry
        sh "aws --region $awsRegion ecr batch-delete-image --image-ids imageTag=$prImageTag --repository-name $imageName"
        sh "aws --region $awsRegion ecr batch-delete-image --image-ids imageTag=$prImageTag --repository-name $imageNameDevelopment"

        // Delete the PR images from registry
        sh "aws --region $awsRegion ecr batch-delete-image --image-ids imageDigest=$prImageDigest --repository-name $imageName"
        sh "aws --region $awsRegion ecr batch-delete-image --image-ids imageDigest=$prImageDigestDevelopment --repository-name $imageNameDevelopment"
      }
    }

    defraUtils.setGithubStatusSuccess()
  } catch(e) {
    defraUtils.setGithubStatusFailure(e.message)
    throw e
  }
}
