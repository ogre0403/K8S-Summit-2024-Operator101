# 生成 CR API

## 建立目錄

```bash
export CRD_NAME=myweb
export GROUP=operator.k8s-summit.org
export VERSION=v1
export BASE_PATH=web-crd

cd /workspaces/K8S-Summit-2024-Operator101/02-CRD-GO-CLIENTSET/${BASE_PATH}


mkdir hack
touch hack/boilerplate.go.txt

mkdir -p pkg/apis/${CRD_NAME}/${VERSION}

```

### 建立types.go 與 doc.go

```bash

cd /workspaces/K8S-Summit-2024-Operator101/02-CRD-GO-CLIENTSET/${BASE_PATH}

# 建立 doc.go

cat << EOF > pkg/apis/${CRD_NAME}/${VERSION}/doc.go
// +k8s:deepcopy-gen=package
// +groupName=${GROUP}
package ${VERSION}
EOF
```

```bash
cd /workspaces/K8S-Summit-2024-Operator101/02-CRD-GO-CLIENTSET/${BASE_PATH}

# 建立 types.go

cat << EOF > pkg/apis/${CRD_NAME}/${VERSION}/types.go
package ${VERSION}

import (
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
)

// +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object
// +genclient
type MyWeb struct {
	metav1.TypeMeta   \`json:",inline"\`
	metav1.ObjectMeta \`json:"metadata,omitempty"\`

	Spec   MyWebSpec   \`json:"spec"\`
	Status MyWebStatus \`json:"status"\`
}

type MyWebSpec struct {
	Image           string \`json:"image"\`
	NodePortNumber  int    \`json:"nodePortNumber"\`
	PageContentHtml string \`json:"pageContentHtml"\`
}

type MyWebStatus struct {
	Completed bool \`json:"completed"\`
}

// +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object
type MyWebList struct {
	metav1.TypeMeta \`json:",inline"\`
	metav1.ListMeta \`json:"metadata,omitempty"\`

	Items []MyWeb \`json:"items"\`
}

EOF
```


## Install code generators

```bash
cd /workspaces/K8S-Summit-2024-Operator101/02-CRD-GO-CLIENTSET/${BASE_PATH}

go mod tidy

make auto-generate
```

## 使用 clientset

```bash
cd /workspaces/K8S-Summit-2024-Operator101/02-CRD-GO-CLIENTSET/${BASE_PATH}

cat << EOF > main.go
package main

import (
	"context"
	"os"
	"path"
	"${BASE_PATH}/pkg/clientset"

	webv1 "${BASE_PATH}/pkg/apis/${CRD_NAME}/${VERSION}"

	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/client-go/tools/clientcmd"
)

func main() {

	home, err := os.UserHomeDir()
	if err != nil {
		panic(err)
	}

	config, err := clientcmd.BuildConfigFromFlags("", path.Join(home, ".kube/config"))
	if err != nil {
		panic(err.Error())
	}

	clientset, err := clientset.NewForConfig(config)
	if err != nil {
		panic(err)
	}

	contentHtml := \`
<html>
	<body>
		<h1>Hello, World!</h1>
	</body>
</html>\`

	myresource := &webv1.MyWeb{
		ObjectMeta: metav1.ObjectMeta{
			Name: "myresource",
		},
		Spec: webv1.MyWebSpec{
			Image:           "nginx",
			NodePortNumber:  30100,
			PageContentHtml: contentHtml,
		},
	}
	clientset.OperatorV1().MyWebs("default").Create(context.Background(), myresource, metav1.CreateOptions{})

	_, err = clientset.OperatorV1().MyWebs("default").Get(context.Background(), "myresource", metav1.GetOptions{})
	if err != nil {
		panic(err)
	}

}
EOF
```

```bash
cd /workspaces/K8S-Summit-2024-Operator101/02-CRD-GO-CLIENTSET/${BASE_PATH}

cat << EOF > go.mod
module web-crd

go 1.25

require (
	k8s.io/apimachinery v0.35.5
	k8s.io/client-go v0.35.5
)
EOF
```

```bash
cd /workspaces/K8S-Summit-2024-Operator101/02-CRD-GO-CLIENTSET/${BASE_PATH}

go mod tidy

go build
```


## Reference
1. [最不厌其烦的 K8s 代码生成教程](https://www.zeng.dev/post/2023-k8s-api-codegen/)
2. [code-generator简单介绍](https://juejin.cn/post/7096484178128011277)