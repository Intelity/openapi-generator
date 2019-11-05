@Library("release-pipeline") _

pipeline {
  agent {
    docker {
      image 'registry.keypr.com/build-android:latest'
      registryUrl env.DOCKER_REGISTRY_URL
      registryCredentialsId 'keyprbuild-ldap-credentials'
    }
  }

  options {
    disableConcurrentBuilds()
    timeout(time: 30, unit: 'MINUTES')
  }

  parameters {
    booleanParam(
        name: 'Publish',
        description: 'Check this flag if you want to publish resulted artifacts',
        defaultValue: false,
    )
  }

  stages {
    stage('Post-checkout') {
      steps {
        gitCheckCiSkip()
      }
    }

    stage('Build PR') {
      when {
        changeRequest()
      }
      steps {
        sh './gradlew :openapi-generator-gradle-plugin-mvn-wrapper:clean build check --refresh-dependencies --no-daemon --stacktrace'
      }
      post {
        always {
          androidLint pattern: '*/build/reports/lint-*.xml'
          slackNotify('kaos-slack-notifier-api-token', currentBuild.result, ['FIXED', 'BROKEN', 'UNSTABLE'])
        }
      }
    }

    stage('Build master') {
      when {
        branch 'master'
      }
      steps {
        withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'kaos-repo-writer', usernameVariable: 'KEYPR_USER', passwordVariable: 'KEYPR_PASSWORD']]) {
          sh '''#!/bin/bash
            extra=""
            if [ "$Publish" = "true" ]
            then
              extra="upload"
            fi
            ./gradlew :openapi-generator-gradle-plugin-mvn-wrapper:clean build $extra --no-daemon --stacktrace
          '''
        }
      }
      post {
        always {
          androidLint pattern: '*/build/reports/lint-*.xml'
          slackNotify('kaos-slack-notifier-api-token', currentBuild.result, ['FIXED', 'BROKEN', 'UNSTABLE'])
        }
      }
    }
  }

  post {
    always {
      deleteDir()
    }
  }
}
