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
                                      --destination=naveenkumar003/myapp:${BUILD_NUMBER}
                        '''
                  }
               }
               
        }
      }
     stage("Deployment") {
          steps {
                git branch: 'main', url: 'https://github.com/naveenkumarhn/Jenkins.git'
               sh '''
                  git config --global user.email "nkumar1805@yahoo.in"
                  git config --global user.name naveenkumarhn
                  sed -i "s/alpine-webserver:.*/webapp:${BUILD_NUMBER}/g" deployment/deployment.yaml
                  git commit -am "${BUILD_NUMBER}"
                  ls
                  git push --force origin master
                 '''
        }  
       }
     }
}
