# Extensions
load('ext://podman', 'podman_build')
load('ext://dotenv', 'dotenv')

# Load environment file
dotenv()

# Set the default registry
default_registry(os.getenv('REGISTRY'))

# Allow deployment to default namespace
allow_k8s_contexts('default')

# Helper function to convert objects to flattened list of pairs
def flatten(o, p=''):
  l = []
  if type(o) == 'list':
    for i, e in enumerate(o):
      lk = '{}[{}]'.format(p, i) if p else ''
      l.extend(flatten(e, lk))
  elif type(o) == 'dict':
    for k, v in o.items():
      k = k.replace('.', '\\.') if k.find('.') != -1 else k
      dk = '{}.{}'.format(p, k) if p else k
      l.extend(flatten(v, dk))
  else:
    e = '{}={}'.format(p, o) if p else o
    l.append(e)
  return l

# Iterate on all projects
projects = os.getenv('PROJECTS').split(',')
for p in projects:
  # Deploy: tell Tilt what YAML to deploy
  host = '{}.{}'.format(p, os.getenv('HOST'))
  values = {
    'ingress': {
      'enabled': True,
      'annotations': {'cert-manager.io/cluster-issuer': 'local-ca-issuer'},
      'hosts': [{
        'host': host,
        'paths': [{
          'path': '/',
          'pathType': 'ImplementationSpecific'
        }]
      }],
      'tls': [{
        'secretName': '{}-tls'.format(p),
        'hosts': [host]
      }]
    }
  }
  chart_dir = os.path.join(p, 'chart')
  yaml = helm(chart_dir, name=p, set=flatten(values))
  k8s_yaml(yaml)

  # Build: tell Tilt what images to build from which directories
  podman_build(p, p, extra_flags=['--arch', os.getenv('ARCH')])
