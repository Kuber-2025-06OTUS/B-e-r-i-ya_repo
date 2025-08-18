Подготовка
- minikube start
- minikube addons enable ingress
- kubectl label nodes minikube homework="true"
- cd kubernetes-security
- kubectl apply -f namespace.yaml
- kubectl create serviceaccount monitoring -n homework

- create mr-clusterrole.yaml
```
cat <<EOF > mr-clusterrole.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: metrics-reader
rules:
- nonResourceURLs: ["/metrics"]
  verbs: ["get"]
EOF
```
- kubectl apply -f mr-clusterrole.yaml

- create mr-clusterrolebinding.yaml
```
cat <<EOF > mr-clusterrolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: monitoring-metrics-reader
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: metrics-reader
subjects:
- kind: ServiceAccount
  name: monitoring
  namespace: homework
EOF
```

- kubectl apply -f mr-clusterrolebinding.yaml

- create secret_SA.yaml
```
cat <<EOF > secret_SA.yaml
apiVersion: v1
kind: Secret
metadata:
  name: monitoring-token
  namespace: homework
  annotations:
    kubernetes.io/service-account.name: monitoring
type: kubernetes.io/service-account-token
EOF
```
- kubectl apply -f secret_SA.yaml

- kubectl get secret -n homework monitoring-token -o jsonpath='{.data.token}' | base64 --decode

- kubectl cluster-info | grep 'Kubernetes control plane' | awk '{print $NF}'

- API_SERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')

- TOKEN=$(kubectl get secret -n homework monitoring-token -o jsonpath='{.data.token}' | base64 --decode)

- curl -k -H "Authorization: Bearer $TOKEN" "$API_SERVER/metrics"


- kubectl apply -f StoreClass.yaml
- kubectl apply -f pvc.yaml
- kubectl apply -f cm.yaml
- kubectl apply -f deployment.yaml
- kubectl apply -f service.yaml
- kubectl apply -f ingress.yaml
- echo $(minikube ip) homework.otus >> /etc/hosts

Проверяем
- kubectl get pods -n homework -o yaml | grep serviceAccountName

- kubectl apply -f cd-admin-RB.yaml

Проверяем
- kubectl get rolebinding cd-admin -n homework -o yaml

- kubectl apply -f cd-secret.yaml
- APISERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')
- CLUSTER_NAME=$(kubectl config view --minify -o jsonpath='{.clusters[0].name}')
- TOKEN=$(kubectl get secret cd-token -n homework -o jsonpath='{.data.token}' | base64 --decode)

- kubectl get secret cd-token -n homework -o jsonpath='{.data.ca\.crt}' | base64 --decode > ca.crt

```
cat <<EOF > cd-kubeconfig.yaml
apiVersion: v1
kind: Config
clusters:
- name: ${CLUSTER_NAME}
  cluster:
    certificate-authority-data: $(cat ca.crt | base64 | tr -d '\n')
    server: ${APISERVER}
users:
- name: cd
  user:
    token: ${TOKEN}
contexts:
- name: cd@${CLUSTER_NAME}
  context:
    cluster: ${CLUSTER_NAME}
    user: cd
    namespace: homework
current-context: cd@${CLUSTER_NAME}
EOF
```

- KUBECONFIG=cd-kubeconfig.yaml kubectl get pods -n homework(kubectl --kubeconfig=cd-kubeconfig.yaml get pods)

- kubectl apply -f cd-token.yaml
- kubectl get secret cd-token-expiring -n homework -o jsonpath='{.data.token}' | base64 --decode > token

- APISERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')
- TOKEN=$(cat token)
- curl -k -H "Authorization: Bearer $TOKEN" "$APISERVER/api/v1/namespaces/homework/pods"


Задание со *
- kubectl delete -f deployment.yaml
- kubectl apply -f deployment_new.yaml

