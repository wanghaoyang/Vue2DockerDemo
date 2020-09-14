pipeline {
  agent {
    node {
      label 'nodejs'
    }

  }
  stages {
    stage('拉取代码') {
      steps {
        git(url: 'https://github.com/wanghaoyang/Vue2DockerDemo.git', credentialsId: 'github-id', branch: 'master', changelog: true, poll: false)
      }
    }
    stage('构建镜像 & 推送快照镜像') {
      steps {
        container('nodejs') {
          sh 'docker build -f dockerfile -t $REGISTRY/$DOCKERHUB_NAMESPACE/$PROJECT_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER .'
          withCredentials([usernamePassword(credentialsId : 'dockerhub-id' ,passwordVariable : 'DOCKER_PASSWORD' ,usernameVariable : 'DOCKER_USERNAME' ,)]) {
            sh 'echo "$DOCKER_PASSWORD" | docker login $REGISTRY -u "$DOCKER_USERNAME" --password-stdin'
            sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$PROJECT_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER'
          }

        }

      }
    }
    stage('推送最新镜像') {
      when {
        branch 'master'
      }
      steps {
        container('nodejs') {
          sh 'docker tag  $REGISTRY/$DOCKERHUB_NAMESPACE/$PROJECT_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER $REGISTRY/$DOCKERHUB_NAMESPACE/$PROJECT_NAME:latest '
          sh 'docker push $REGISTRY/$DOCKERHUB_NAMESPACE/$PROJECT_NAME:latest '
          sh 'docker images'
        }

      }
    }
    stage('部署到 dev 环境') {
      when {
        branch 'master'
      }
      steps {
        input(message: "deploy $PROJECT_NAME to dev?", submitter: '')
        kubernetesDeploy(enableConfigSubstitution: true, deleteResource: false, kubeconfigId: '$KUBECONFIG_CREDENTIAL_ID', configs: 'k8s-deploy.yaml')
      }
    }
    stage('发布版本') {
      when {
        expression {
          params.PROJECT_VERSION =~ /v.*/
        }

      }
      steps {
        container('maven') {
          input(message: 'release $PROJECT_NAME image with version?', submitter: '')
          withCredentials([usernamePassword(credentialsId : '$GITHUB_CREDENTIAL_ID' ,passwordVariable : 'GIT_PASSWORD' ,usernameVariable : 'GIT_USERNAME' ,)]) {
            sh 'git config --global user.email "myemail@mxmx.com" '
            sh 'git config --global user.name "myname" '
            sh 'git tag -a $PROJECT_VERSION -m "$PROJECT_VERSION" '
            sh 'git push https://$GIT_USERNAME:$GIT_PASSWORD@github.com/$GITHUB_ACCOUNT/devops-java-sample.git --tags --ipv4'
          }

          sh 'docker tag  $REGISTRY/$DOCKERHUB_NAMESPACE/$PROJECT_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER $REGISTRY/$DOCKERHUB_NAMESPACE/$PROJECT_NAME:$PROJECT_VERSION '
          sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$PROJECT_NAME:$PROJECT_VERSION '
        }

      }
    }
    stage('部署到 prod 环境') {
      when {
        expression {
          params.PROJECT_VERSION =~ /v.*/
        }

      }
      steps {
        input(message: "deploy $PROJECT_NAME to production?", submitter: '')
        kubernetesDeploy(enableConfigSubstitution: true, deleteResource: false, kubeconfigId: '$KUBECONFIG_CREDENTIAL_ID', configs: 'deploy/prod-ol/**')
      }
    }
  }
  environment {
    DOCKER_CREDENTIAL_ID = 'dockerhub-id'
    GITHUB_CREDENTIAL_ID = 'github-id'
    KUBECONFIG_CREDENTIAL_ID = 'demo-kubeconfig'
    REGISTRY = 'docker.io'
    DOCKERHUB_NAMESPACE = 'hy29108'
    GITHUB_ACCOUNT = 'wanghaoyang'
    SONAR_CREDENTIAL_ID = 'sonar-qube-token'
    GITEE_ACCOUNT = 'xxxxx'
    ALIYUN_DOCKER_CREDENTIAL_ID = 'aliyun-dockerhub-id'
    ALIYUN_REGISTRY = 'registry.cn-hangzhou.aliyuncs.com'
    ALIYUNHUB_NAMESPACE = 'xxxxxx'
  }
  parameters {
    string(name: 'PROJECT_NAME', defaultValue: 'my-app', description: '项目名称')
    string(name: 'PROJECT_VERSION', defaultValue: '', description: '项目版本')
  }
}
