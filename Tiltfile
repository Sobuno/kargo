if 'ENABLE_NGROK_EXTENSION' in os.environ and os.environ['ENABLE_NGROK_EXTENSION'] == '1':
  v1alpha1.extension_repo(
    name = 'default',
    url = 'https://github.com/tilt-dev/tilt-extensions'
  )
  v1alpha1.extension(name = 'ngrok', repo_name = 'default', repo_path = 'ngrok')

trigger_mode(TRIGGER_MODE_MANUAL)

load('ext://namespace', 'namespace_create')
namespace_create('k8sta')
k8s_resource(
  new_name = 'common',
  objects = [
    'k8sta:namespace',
    'k8sta-config:configmap'
  ],
  labels = ['k8sta']
)

docker_build(
  'akuity/k8sta',
  '.',
  only = [
    'cmd/',
    'internal/',
    'go.mod',
    'go.sum',
    'main.go'
  ],
  ignore = ['**/*_test.go']
)
k8s_resource(
  workload = 'k8sta-server',
  new_name = 'server',
  port_forwards = '30081:8080',
  labels = ['k8sta']
)
k8s_resource(
  workload = 'server',
  objects = [
    'k8sta-server:clusterrole',
    'k8sta-server:clusterrolebinding',
    'k8sta-server:serviceaccount',
    'k8sta-server-config:secret'
  ]
)
k8s_resource(
  workload = 'k8sta-controller',
  new_name = 'controller',
  labels = ['k8sta']
)
k8s_resource(
  workload = 'controller',
  objects = [
    'k8sta-controller:clusterrole',
    'k8sta-controller:clusterrolebinding',
    'k8sta-controller:serviceaccount'
  ]
)

k8s_yaml(
  helm(
    './charts/k8sta',
    name = 'k8sta',
    namespace = 'k8sta',
    set = [
      'server.tls.enabled=false',
      'server.dockerhub.tokens.dev-token=insecure-dev-token'
    ]
  )
)