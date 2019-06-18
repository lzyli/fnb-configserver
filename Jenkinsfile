pipeline {
  agent {
    label "jenkins-maven"
  }
  parameters {
        string(name: 'tag', defaultValue: '')
  }
  environment {
    ORG = 'tuananhho'
    APP_NAME = 'fnb-configserver'
    CHARTMUSEUM_CREDS = credentials('jenkins-x-chartmuseum')
    DOCKER_REGISTRY_ORG = 'tuananhho'
  }
   stages {
    stage('CI Build and push snapshot') {
      when {
        branch 'PR-*'
      }
      environment {
        PREVIEW_VERSION = "0.0.0-SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER"
        PREVIEW_NAMESPACE = "$APP_NAME-$BRANCH_NAME".toLowerCase()
        HELM_RELEASE = "$PREVIEW_NAMESPACE".toLowerCase()
        // TAG = ${params.tag}
      }
      steps {
        container('maven') {
          // sh "mvn versions:set -DnewVersion=$PREVIEW_VERSION"
          // sh "mvn install"
          // sh "skaffold version"
          // sh "export VERSION=$PREVIEW_VERSION && skaffold build -f skaffold.yaml"
          // sh "jx step post build --image $DOCKER_REGISTRY/$ORG/$APP_NAME:$PREVIEW_VERSION"
          // dir('charts/preview') {
          //   sh "make preview"
          //   sh "jx preview --app $APP_NAME --dir ../.."
          }
        }
      }
    stage('Build Release') {
      when {
        branch 'master'
      }
      steps {
        container('maven') {
          // ensure we're not on a detached head
          sh "tag=${params.tag}"
          sh "echo $tag"
          sh "git checkout master"
          sh "git config --global credential.helper store"
          sh "jx step git credentials"
          // so we can retrieve the version in later steps
          sh "echo \$(jx-release-version) > VERSION"
          // sh "mvn versions:set -DnewVersion=\$(cat VERSION)"
          sh "jx step tag --version \$(cat VERSION)"
          // sh "mvn clean deploy"
          sh "skaffold version"
          sh "export VERSION=`cat VERSION` && skaffold build -f skaffold.yaml"
          // sh "jx step post build --image $DOCKER_REGISTRY/$ORG/$APP_NAME:\$(cat VERSION)"
        }
      }
    }
    stage('Promote to Environments') {
      when {
        branch 'master'
      }
      steps {
        container('maven') {
          dir('charts/fnb-configserver') {
            // sh 'tag=${params.tag}'
            sh "sed -i -e 's/sptag:.*/sptag: $(tag)' values.yaml"
            sh "jx step changelog --version v\$(cat ../../VERSION)"

            // release the helm chart
            sh "jx step helm release"

            // promote through all 'Auto' promotion Environments
            //  sh "jx promote -b --all-auto --timeout 1h --version \$(cat ../../VERSION)"
            sh "jx step helm apply --namespace default"
          }
        }
      }
    }
   }
  post {
        always {
          cleanWs()
        }
  }
}
