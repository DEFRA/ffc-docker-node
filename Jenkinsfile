@Library('defra-library@0.0.8')
import uk.gov.defra.ffc.DefraUtils
def defraUtils = new DefraUtils()

def registry = '562955126301.dkr.ecr.eu-west-2.amazonaws.com'
def repoName = 'ffc-docker-parent'
def regCredsId = 'ecr:eu-west-2:ecr-user'
def pr = ''
def mergedPrNo = ''
def containerTag = ''
def nodeVersions = ['10.19.0','12.16.0']
def version = '1.0.0'
def tagVersion = version
def imageName = 'ffc-node-parent'
def devImageName = 'ffc-node-development'

node {
  checkout scm
  try {
    stage('Set branch, PR, and containerTag variables') {
      (pr, containerTag, mergedPrNo) = defraUtils.getVariables(repoName)
      defraUtils.setGithubStatusPending()
    }
    stage('Build Parent image') {
      // Build the parent image. 1 parent image per node version required.
      // Tag the images with the PR build (if it's a PR build)
      // Name the image according to the node version used
      // Tag the image according to our version, if it's PR build, include the PR number in the version such as @1.0.0-pr1
      tagVersion = version
      if (pr) {
        tagVersion = tagVersion + "-pr${pr}"
      }
      imageName = "ffc-node-parent-${nodeVersions[0]}"
      sh "docker build --no-cache --tag $imageName:$tagVersion --tag $registry/$imageName:$tagVersion--build-arg NODE_VERSION=${nodeVersions[0]} \
      --build-arg VERSION=$version ffc-node-parent/. "
    }
    stage('Build development image') {
      devImageName = "ffc-node-development-${nodeVersions[0]}"
      sh "docker build --no-cache --tag $devImageName:$tagVersion --tag $registry/$devImageName:$tagVersion --build-arg PARENT_IMAGE=$imageName \
      --build-arg BASE_VERSION=$version --build-arg REGISTRY=$registry ffc-node-development/. "
    }

    // Then build the dev images, 1 per node version that reference the parent image that has that node version
    // Name those according to the node version used
    // Tag them with our version
    // Push the parent images
    // Push the development images
    // If this is a merge to master, delete the PR images


    defraUtils.setGithubStatusSuccess()
  } catch(e) {
    defraUtils.setGithubStatusFailure(e.message)
    throw e
  }
}
