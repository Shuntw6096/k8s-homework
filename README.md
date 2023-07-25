# k8s-homework
How to use the `curl` command be used in a Pod to retrieve all Pods in the `default` namespace?


# 1.  配置Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: curl-pod
  namespace: default
spec:
  containers:
  - name: curl-container
    image: curlimages/curl:latest
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    command: ["sleep", "infinity"]
```
我使用```curlimages/curl```當作映像檔, 自帶```curl```指令. 這個pod的namespace是default.

為了存取namespace ```default```底下pod的資訊, 我需要創建一個role, 它擁有可以查看pod的資訊的權限.

# 2. 配置role查看namespace的pod資源
這個role的名稱是```pod-reader```, 在每個命名空間都會有預設的ServiceAccount ```default```, ```default```命名空間的預設ServiceAccount是```default:default```.  

然就要把```pod-reader```綁定到```default:default```.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```


# 3. 綁定role才可以查看資源
使用```rbac.authorization.k8s.io```的API底下的object ```RoleBinding```把```pod-reader```綁定到```default:default```.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
  namespace: default
subjects:
- kind: ServiceAccount
  name: default
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```



# 執行方式


```bash
kubectl apply -f ./role.yaml
kubectl apply -f ./rolebinding.yaml
kubectl create -f ./pod.yaml
kubectl exec -it curl-pod -- sh

curl -k -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" https://kubernetes.default.svc/api/v1/namespaces/default/pods
```
輸出是
```json
~ $ curl -k -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/servi
ceaccount/token)" https://kubernetes.default.svc/api/v1/namespaces/default/pods


{
  "kind": "PodList",
  "apiVersion": "v1",
  "metadata": {
    "resourceVersion": "774"
  },
  "items": [
    {
      "metadata": {
        "name": "curl-pod",
        "namespace": "default",
        "uid": "4da175b9-74c9-4c4a-9209-a230996b843a",
        "resourceVersion": "687",
        "creationTimestamp": "2023-07-25T15:12:52Z",
        "managedFields": [
          {
            "manager": "kubectl-create",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2023-07-25T15:12:52Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:spec": {
                "f:containers": {
                  "k:{\"name\":\"curl-container\"}": {
                    ".": {},
                    "f:command": {},
                    "f:image": {},
                    "f:imagePullPolicy": {},
                    "f:name": {},
                    "f:resources": {
                      ".": {},
                      "f:limits": {
                        ".": {},
                        "f:cpu": {},
                        "f:memory": {}
                      },
                      "f:requests": {
                        ".": {},
                        "f:cpu": {},
                        "f:memory": {}
                      }
                    },
                    "f:terminationMessagePath": {},
                    "f:terminationMessagePolicy": {}
                  }
                },
                "f:dnsPolicy": {},
                "f:enableServiceLinks": {},
                "f:restartPolicy": {},
                "f:schedulerName": {},
                "f:securityContext": {},
                "f:terminationGracePeriodSeconds": {}
              }
            }
          },
          {
            "manager": "kubelet",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2023-07-25T15:12:55Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:status": {
                "f:conditions": {
                  "k:{\"type\":\"ContainersReady\"}": {
                    ".": {},
                    "f:lastProbeTime": {},
                    "f:lastTransitionTime": {},
                    "f:status": {},
                    "f:type": {}
                  },
                  "k:{\"type\":\"Initialized\"}": {
                    ".": {},
                    "f:lastProbeTime": {},
                    "f:lastTransitionTime": {},
                    "f:status": {},
                    "f:type": {}
                  },
                  "k:{\"type\":\"Ready\"}": {
                    ".": {},
                    "f:lastProbeTime": {},
                    "f:lastTransitionTime": {},
                    "f:status": {},
                    "f:type": {}
                  }
                },
                "f:containerStatuses": {},
                "f:hostIP": {},
                "f:phase": {},
                "f:podIP": {},
                "f:podIPs": {
                  ".": {},
                  "k:{\"ip\":\"10.244.0.12\"}": {
                    ".": {},
                    "f:ip": {}
                  }
                },
                "f:startTime": {}
              }
            },
            "subresource": "status"
          }
        ]
      },
      "spec": {
        "volumes": [
          {
            "name": "kube-api-access-mvpq5",
            "projected": {
              "sources": [
                {
                  "serviceAccountToken": {
                    "expirationSeconds": 3607,
                    "path": "token"
                  }
                },
                {
                  "configMap": {
                    "name": "kube-root-ca.crt",
                    "items": [
                      {
                        "key": "ca.crt",
                        "path": "ca.crt"
                      }
                    ]
                  }
                },
                {
                  "downwardAPI": {
                    "items": [
                      {
                        "path": "namespace",
                        "fieldRef": {
                          "apiVersion": "v1",
                          "fieldPath": "metadata.namespace"
                        }
                      }
                    ]
                  }
                }
              ],
              "defaultMode": 420
            }
          }
        ],
        "containers": [
          {
            "name": "curl-container",
            "image": "curlimages/curl:latest",
            "command": [
              "sleep",
              "infinity"
            ],
            "resources": {
              "limits": {
                "cpu": "500m",
                "memory": "128Mi"
              },
              "requests": {
                "cpu": "500m",
                "memory": "128Mi"
              }
            },
            "volumeMounts": [
              {
                "name": "kube-api-access-mvpq5",
                "readOnly": true,
                "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount"
              }
            ],
            "terminationMessagePath": "/dev/termination-log",
            "terminationMessagePolicy": "File",
            "imagePullPolicy": "Always"
          }
        ],
        "restartPolicy": "Always",
        "terminationGracePeriodSeconds": 30,
        "dnsPolicy": "ClusterFirst",
        "serviceAccountName": "default",
        "serviceAccount": "default",
        "nodeName": "minikube",
        "securityContext": {},
        "schedulerName": "default-scheduler",
        "tolerations": [
          {
            "key": "node.kubernetes.io/not-ready",
            "operator": "Exists",
            "effect": "NoExecute",
            "tolerationSeconds": 300
          },
          {
            "key": "node.kubernetes.io/unreachable",
            "operator": "Exists",
            "effect": "NoExecute",
            "tolerationSeconds": 300
          }
        ],
        "priority": 0,
        "enableServiceLinks": true,
        "preemptionPolicy": "PreemptLowerPriority"
      },
      "status": {
        "phase": "Running",
        "conditions": [
          {
            "type": "Initialized",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2023-07-25T15:12:52Z"
          },
          {
            "type": "Ready",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2023-07-25T15:12:55Z"
          },
          {
            "type": "ContainersReady",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2023-07-25T15:12:55Z"
          },
          {
            "type": "PodScheduled",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2023-07-25T15:12:52Z"
          }
        ],
        "hostIP": "192.168.49.2",
        "podIP": "10.244.0.12",
        "podIPs": [
          {
            "ip": "10.244.0.12"
          }
        ],
        "startTime": "2023-07-25T15:12:52Z",
        "containerStatuses": [
          {
            "name": "curl-container",
            "state": {
              "running": {
                "startedAt": "2023-07-25T15:12:55Z"
              }
            },
            "lastState": {},
            "ready": true,
            "restartCount": 0,
            "image": "curlimages/curl:latest",
            "imageID": "docker-pullable://curlimages/curl@sha256:daf3f46a2639c1613b25e85c9ee4193af8a1d538f92483d67f9a3d7f21721827",
            "containerID": "docker://c59c254e3aeded02ef86decab2b0e0a9abc9b2ea4564861df01fb842870ead5f",
            "started": true
          }
        ],
        "qosClass": "Guaranteed"
      }
    }
  ]
}~ $
```
