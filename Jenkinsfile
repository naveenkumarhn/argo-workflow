pipeline {

   agent {
    kubernetes {
        yaml """
apiVersion: v1
kind: Pod
metadata:
  name: kaniko
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command:
    - /busybox/sh
    tty: true
    volumeMounts:
    - name: kaniko-secret
      mountPath: /kaniko/.docker
  volumes:
    - name: kaniko-secret
      secret:
        secretName: regcred
        items:
          - key: .dockerconfigjson
            path: config.json
   
"""
   }
     }
     stages {
     stage("Build and Publish") {
         steps {
               container('kaniko') {
                  script {
                     sh '''
                     /kaniko/executor --dockerfile `pwd`/Dockerfile \
                                      --context `pwd` \
                                      --destination=poojansds/webapp:${BUILD_NUMBER}
                        '''
                  }
               }
               
        }
      }
     stage("Deployment") {
          steps {
             sshagent(credentials: ["github-ssh-key"]) {
                git branch: 'master',credentialsId: 'github-ssh-key', url: 'git@github.com:NarayanPooja/aargocd.git'
                
               sh '''
                  git config --global user.email "poojanarayan0805@gmail.com"
                  git config --global user.name NarayanPooja
                  sed -i "s/webapp:.*/webapp:${BUILD_NUMBER}/g" deployment/deployment.yaml
                  git commit -am "${BUILD_NUMBER}"
                  ls
                  git push --force origin master
                 '''
        } 
          }
       }
     }
}
