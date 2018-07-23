node {
  stage("Reconfigure the namespace") {
    checkout scm
    sh "oc apply -f config/fedora-imagestream.yaml"
    sh "oc apply -f config/imagestream.yaml"
    sh "oc apply -f config/buildconfig.yaml"
    sh "oc apply -f config/deploymentconfig.yaml"
    sh "oc apply -f config/service.yaml"
    sh "oc apply -f config/deploymentconfig-tested.yaml"
    sh "oc apply -f config/service-tested.yaml"
    sh "oc apply -f config/route.yaml"
  }

  stage("Build") {
      openshiftBuild buildConfig: "pipeline-app", showBuildLogs: "true"
  }

  stage("Deploy to dev") {
      openshiftDeploy deploymentConfig: "pipeline-app"
  }

  stage("Smoketest") {
    def prefix = "http://pipeline-app.pipelines.svc:8080"

    sh "curl -kLvs ${prefix}/ | grep 'Hello, Anonymous'"
    sh "curl -kLvs ${prefix}/containers | grep 'Hello, containers'"
    sh "curl -kLvs ${prefix}/Minsk | grep 'Hello, Minsk'"
  }

  stage("Deploy to tested") {
      openshiftTag srcStream: "pipeline-app", srcTag: 'latest', destinationStream: "pipeline-app", destinationTag: "smoketested"
      openshiftDeploy deploymentConfig: "pipeline-app-tested"
  }
}
