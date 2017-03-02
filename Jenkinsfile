#!/usr/bin/groovy
@Library('github.com/fabric8io/fabric8-pipeline-library@v2.2.311')

def failIfNoTests = ""
try {
  failIfNoTests = ITEST_FAIL_IF_NO_TEST
} catch (Throwable e) {
  failIfNoTests = "false"
}

def localItestPattern = ""
try {
  localItestPattern = ITEST_PATTERN
} catch (Throwable e) {
  localItestPattern = "*KT"
}


def versionPrefix = ""
try {
  versionPrefix = VERSION_PREFIX
} catch (Throwable e) {
  versionPrefix = "1.0"
}

def canaryVersion = "${versionPrefix}.${env.BUILD_NUMBER}"
def utils = new io.fabric8.Utils()
def label = "buildpod.${env.JOB_NAME}.${env.BUILD_NUMBER}".replace('-', '_').replace('/', '_')

mavenNode{
  def envStage = utils.environmentNamespace('staging')

  git 'http://gogs/gogsadmin/voxxed-age-checker.git'

  echo 'NOTE: running pipelines for the first time will take longer as build and base docker images are pulled onto the node'
  container(name: 'maven') {

    stage 'Fixing Release'
    sh 'mvn walkmod:patch'
    if (fileExists('walkmod.patch')) {
      echo 'walkmod has produced a patch'
      sh 'git apply walkmod.patch'
      sh 'git commit -a --ammend -m "Fixing style violations"'
      sh 'git push'
      currentBuild.result = 'FAILURE'
      error("Build failed by the lack of consistent coding style")

    }
    echo 'no pending quick fixes to apply'
    stage 'Build Release'
    mavenCanaryRelease {
      version = canaryVersion
    }

    stage 'Integration Test'
    mavenIntegrationTest {
      environment = 'Testing'
      failIfNoTests = localFailIfNoTests
      itestPattern = localItestPattern
    }

    stage 'Rollout Staging'
    kubernetesApply(environment: envStage)

  }
}
