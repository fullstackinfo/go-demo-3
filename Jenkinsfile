import java.text.SimpleDateFormat

currentBuild.displayName = new SimpleDateFormat("yy.MM.dd").format(new Date()) + "-" + env.BUILD_NUMBER // NEW!!!
env.PROJECT = "go-demo-3"
env.IMAGE = "vfarcic/go-demo-3" 
env.DOMAIN = "acme.com"
env.ADDRESS = "go-demo-3.acme.com"
env.CM_ADDR = "acme.com"
env.CHART_VER = "0.0.1"

podTemplate(
  label: "kubernetes",
  namespace: "go-demo-3-build",
  serviceAccount: "build",
  yaml: """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: helm
    image: vfarcic/helm:2.9.1
    command: ["cat"]
    tty: true
  - name: kubectl
    image: vfarcic/kubectl
    command: ["cat"]
    tty: true
  - name: golang
    image: golang:1.9
    command: ["cat"]
    tty: true
"""
) {
  node("kubernetes") {
    node("docker") {
      stage("build") {
        k8sBuildImageBeta(env.IMAGE)
      }
    }
    stage("func-test") {
      try {
        container("helm") {
          k8sUpgradeBeta(env.PROJECT, env.DOMAIN)
        }
        container("kubectl") {
          k8sRolloutBeta(env.PROJECT)
        }
        container("golang") {
          k8sFuncTestGolang(env.PROJECT, env.DOMAIN)
        }
      } catch(e) {
          error "Failed functional tests"
      } finally {
        container("helm") {
          k8sDeleteBeta(env.PROJECT)
        }
      }
    }
    stage("release") {
      node("docker") {
        k8sPushImage(env.IMAGE)
      }
      container("helm") {
        k8sPushHelm(env.PROJECT, env.CHART_VER, env.CM_ADDR)
      }
    }
    stage("deploy") {
      try {
        container("helm") {
          k8sUpgrade(env.PROJECT, env.ADDRESS)
        }
        container("kubectl") {
          k8sRollout(env.PROJECT)
        }
        container("golang") {
          k8sProdTestGolang(env.ADDRESS)
        }
      } catch(e) {
        container("helm") {
          k8sRollback(env.PROJECT)
        }
      }
    }
  }
}