# Skopeo Jenkins Slave Image

This repository contains Dockerfiles for a Jenkins Slave Docker image intended for use with OpenShift v3 for suing Skopeo to manipulate docker images without having the docker daemon running: https://github.com/projectatomic/skopeo


## Use it in your Jenkins pipeline
```
oc import-image siamaksade/jenkins-slave-skopeo-centos7 --confirm
oc label is jenkins-slave-skopeo-centos7 role=jenkins-slave

In your Jenkinsfile, use a node named jenkins-slave-skopeo-centos7 to run skopeo commands

```
node('jenkins-slave-skopeo-centos7') { 

  def namespace = readFile('/var/run/secrets/kubernetes.io/serviceaccount/namespace').trim()
  def token = readFile('/var/run/secrets/kubernetes.io/serviceaccount/token').trim()
  def ocCmd = "oc --token=${token} --server=${ocpApiServer} --certificate-authority=/run/secrets/kubernetes.io/serviceaccount/ca.crt --namespace=${namespace}"

  stage('Promote Application') {
    sh """
    set +x
    imageRegistry=\$(${ocCmd} get is ${env.APP_NAME}-dev --template='{{ .status.dockerImageRepository }}' | cut -d/ -f1)
    
    strippedNamespace=\$(echo ${namespace} | cut -d/ -f1)
    
    echo "Promoting \${imageRegistry}/${namespace}/${env.APP_NAME} -> \${imageRegistry}/\${strippedNamespace}-prod/${env.APP_NAME}"
    skopeo --tls-verify=false copy --src-creds openshift:${token} --dest-creds openshift:${token} docker://\${imageRegistry}/${namespace}/${env.APP_NAME} docker://\${imageRegistry}/\${strippedNamespace}-prod/${env.APP_NAME}
    """
  }

```

