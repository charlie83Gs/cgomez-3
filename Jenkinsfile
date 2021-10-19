podTemplate(yaml: '''
              kind: Pod
              metadata:
                annotations: {
                  sidecar.istio.io/inject: false
                }
              spec:
                containers:
                - name: kaniko
                  image: gcr.io/kaniko-project/executor:v1.6.0-debug
                  command:
                  - sleep
                  args:
                  - 99d
                - name: kubectl
                  image: bitnami/kubectl:1.22.2
                  securityContext:
                    runAsUser: 0
                  tty: true
                  command:
                  - sleep
                  args:
                  - 99d

'''
  ) {
  node(POD_LABEL) {
    properties([
      pipelineTriggers([[
        $class: 'GenericTrigger',
        token: 'cgomez-3_cd',
        genericVariables: [[key: 'ref', value: '$.ref']],
        regexpFilterText:"\$ref",
        regexpFilterExpression: 'refs/heads/' +  'main'
    ]])])
    stage('Build with Kaniko') {
      git branch: 'main', credentialsId: 'e1766464-427c-45ac-a109-b337468fb540', url: 'https://github.com/charlie83Gs/cgomez-3.git'
      container('kaniko') {
        withCredentials([file(credentialsId: 'cgomez-3-docker', variable: 'FILE')]) {
          sh 'cp $FILE /kaniko/.docker/config.json'
          sh '/kaniko/executor --dockerfile `pwd`/Dockerfile -c `pwd` --destination=charlie83gs/page:latest'
        }
      }
    }
    stage('Deploy to kubernetes') {
      git branch: 'main', credentialsId: 'e1766464-427c-45ac-a109-b337468fb540', url: 'https://github.com/charlie83Gs/cgomez-3.git'
      container('kubectl') {
        withCredentials([file(credentialsId: 'kubernetes-config-file', variable: 'FILE')]) {
          sh 'mkdir -p ~/.kube/'
          sh 'cp $FILE ~/.kube/config'
          sh 'kubectl apply -f page.yaml'
          sh 'kubectl label namespace cgomez-3 istio-injection=enabled --overwrite'
          sh 'kubectl rollout restart -n cgomez-3 deployment/cgomez-3-deployment'
        }
      }
    }
  }
  }
