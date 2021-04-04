# hoow to mounts?
volumeMounts:
  - name: podinfo
    mountPath: /etc/padinfo
    readOnly: false
...
volumes:
  - name: podinfo
    projected:
      sources:
      - downwardAPI:
          items:
            - path: "labels"
              fieldRef:
                fieldPath: metadata.labels

# what will happen?
Labels field of POd will be mounted to /etc/podinfo/labels file

# what downward api support?
1. use fiedlRef can define and use:
spec.nodename - host name
status.hostIP - host ip
metadata.name - pod name
metadata.namespace - pod's namespace
status.podIP - pod's IP
spec.serviceAccountName - pod's service account name
metadata.uid - pod uid
metadata.label['<KEY>'] - specify key label value
metadata.annotations['<KEY>'] - specify key annotation value
metadata.label - pod's all laels
metadata.annotations - pod's all annotations

2. use resourceFieldRef to define:
cpu limit of container
cpu request of container
memory limit of container
memory request of container
