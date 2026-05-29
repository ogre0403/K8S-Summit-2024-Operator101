# 撰寫 Operator

## 運行最簡單的Operator

```bash 
cd /workspaces/K8S-Summit-2024-Operator101/03-OPERATOR/operator
go mod tidy
go build
```

## 撰寫符合業務邏輯的Operator

```bash
cd /workspaces/K8S-Summit-2024-Operator101/03-OPERATOR/operator

sed -i '/Complete(&MyReconciler{})/c\
		Owns(&corev1.ConfigMap{}).\
		Owns(&corev1.Service{}).\
		Owns(&appsv1.Deployment{}).\
		Complete(&WebReconciler{\
			client: mgr.GetClient(),\
			scheme: mgr.GetScheme(),\
		})' main.go

sed -i '/webv1 "operator\/pkg\/apis\/myweb\/v1"/a\
	appsv1 "k8s.io\/api\/apps\/v1"\
	corev1 "k8s.io\/api\/core\/v1"' main.go

```


```bash
cd /workspaces/K8S-Summit-2024-Operator101/03-OPERATOR/operator

cat << EOF > go.mod
go 1.25.0

require (
	k8s.io/api v0.35.5
	k8s.io/apimachinery v0.35.5
	k8s.io/client-go v0.35.5
	sigs.k8s.io/controller-runtime v0.23.3
)
EOF
```


```bash 
cd /workspaces/K8S-Summit-2024-Operator101/03-OPERATOR/operator
go mod tidy
go build
```