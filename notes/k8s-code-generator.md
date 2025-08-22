# k8s-code-generator

**[code-generator](https://github.com/kubernetes/code-generator)**

## Redhat note

[redhat 教程](https://www.redhat.com/en/blog/kubernetes-deep-dive-code-generation-customresources)

`k8s.io/code-generator` 是 Kubernetes 官方提供的代码生成工具集，能够为自定义资源（CRD）自动生成客户端代码，包括 deepcopy 方法、typed clientset、informer 和 lister 等。

所有 kubernetes code-generators 的实现都基于 [k8s.io/gengo](https://github.com/kubernetes/gengo):

> 这些 generators 都需要一个输入包列表（通过 `--input-dirs` 指定），逐个处理其中的类型，并生成相应代码。生成的代码要么输出到输入文件所在的同一目录（如 deepcopy-gen，可通过 `--output-file-base "zz_generated.deepcopy"` 定义文件名）

k8s.io/code-generator 提供了 `generate-groups.sh` ，其封装了针对自定义资源（CustomResources）场景下调用各个生成器所需的复杂参数和细节。项目中，只需在类似 `hack/update-codegen.sh` 的脚本中执行：

```shell
vendor/k8s.io/code-generator/generate-groups.sh all \
  github.com/openshift-evangelist/crd-code-generation/pkg/client \
  github.com/openshift-evangelist/crd-code-generation/pkg/apis \
  example.com:v1
```

该脚本适用于如下包结构的项目：

> `pkg/apis/example.com/v1/` 下包含 API 类型定义（如 `types.go`、`doc.go` 和 `register.go`）
>
> 通过 `+genclient` 和 `+k8s:deepcopy-gen` 等注释标记生成选项 
>
> 包含必要的头文件（如 `boilerplate.go.txt`）并通过 `--go-header-file` 指定

PS: 从 Kubernetes 1.28 alpha 版本开始，`generate-groups.sh` 已被移除，推荐使用 `k8s.io/code-generator/kube_codegen.sh` 替代

### Tags

在 `*.go`文件中添加 tags 来控制 code-generator 的行为，包括：

1. `doc.go` 文件中包声明上方的 Global tags
2. 写在被处理类型的上方的 Local tags

tags 的一般形式为  `// +tag-name` 或 `// +tag-name=value`

**Global Tags**

写在 `doc.go`

一个典型的 `pkg/apis/<apigroup>/<version>/doc.go` 形如：

```go
// +k8s:deepcopy-gen=package,register
// Package v1 is the v1 version of the API.
 // +groupName=example.com
 package v1
```

其中 `// +k8s:deepcopy-gen=package,register` 告诉 `deepcopy-gen` 为该包中的每个类型默认生成深拷贝（deepcopy）方法

不需要的，可以使用 `// +k8s:deepcopy-gen=false`来禁用

> `register` 关键字的作用是将深拷贝方法注册到 `scheme` 中。但从 Kubernetes 1.9 开始，这一机制将被完全移除，因为 `scheme` 将不再负责对 `runtime.Object` 进行深拷贝。

`// +groupName=example.com`  定义了完整的 API 组名

**Local Tags**

写在 API 类型的正上方，或其上方的第二个注释块中

例：

```go
// +genclient

// +genclient:noStatus

// +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object
// Database describes a database.
 type Database struct {
 metav1.TypeMeta `json:",inline"`
 metav1.ObjectMeta `json:"metadata,omitempty"`


 Spec DatabaseSpec `json:"spec"`
 }


// DatabaseSpec is the spec for a Foo resource
 type DatabaseSpec struct {
 User string `json:"user"`
 Password string `json:"password"`
 Encoding string `json:"encoding,omitempty"`
 }


// +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object


// DatabaseList is a list of Database resources
 type DatabaseList struct {
 metav1.TypeMeta `json:",inline"`
 metav1.ListMeta `json:"metadata"`
 Items []Database `json:"items"`
 }
```

 **`runtime.Object and DeepCopyObject`**

这里有一个特殊的 deepcopy tag：

```go
// +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object
```

Kubernetes 1.8 之后 `runtime.Object` 接口新增了一个方法签名 `DeepCopyObject()` ，实现：

```go
func (in *T) DeepCopyObject() runtime.Object {
    if c := in.DeepCopy(); c != nil {
        return c
    } else {
        return nil
    }
}
```

添加 `// +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object` 之后就手动编写这段代码

**`Client-gen Tags`**

上例中有：

```go
// +genclient
// +genclient:noStatus
```

`// +genclient` 告诉 `client-gen` 为此类型生成一个客户端

`// +genclient:noStatus` 表示该类型不使用通过 `/status` 子资源实现的 `spec` 与 `status` 分离机制

对于集群级别（非命名空间）的资源，则需要：

```go
// +genclient:nonNamespaced
```

还可以精细控制 Client 暴露哪些 HTTP 方法：

```go
// +genclient:noVerbs
// +genclient:onlyVerbs=create,delete
// +genclient:skipVerbs=get,list,create,update,patch,delete,deleteCollection,watch
// +genclient:method=Create,verb=create,result=k8s.io/apimachinery/pkg/apis/meta/v1.Status
```

- `noVerbs`：不生成任何标准操作方法（如 Get、List、Create 等）。
- `onlyVerbs=...`：仅生成指定的 HTTP 操作方法。
- `skipVerbs=...`：跳过生成列出的那些操作方法。

> 最后一个 `// +genclient:method=Create,verb=create,result=k8s.io/apimachinery/pkg/apis/meta/v1.Status`  表示此类型生成一个名为 `Create` 的方法，对应 HTTP 动作 `create`，但其返回值不是该 API 类型本身，而是 `metav1.Status` 对象。
>
> 大多数时候意义不大

### 主函数

可以使用原生 kubernetes 客户端，而不用 dynamic client

一个例子：

```go
import (
    ...
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    "k8s.io/client-go/tools/clientcmd"
    examplecomclientset "github.com/openshift-evangelist/crd-code-generation/pkg/client/clientset/versioned"
)

var (
    kubeconfig = flag.String("kubeconfig", "", "kubeconfig 文件路径。仅当在集群外运行时需要。")
    master     = flag.String("master", "", "Kubernetes API 服务器地址。会覆盖 kubeconfig 中的配置。仅当在集群外运行时需要。")
)

func main() {
    flag.Parse()

    cfg, err := clientcmd.BuildConfigFromFlags(*master, *kubeconfig)
    if err != nil {
        glog.Fatalf("构建 kubeconfig 时出错: %v", err)
    }

    exampleClient, err := examplecomclientset.NewForConfig(cfg)
    if err != nil {
        glog.Fatalf("构建 example 客户端时出错: %v", err)
    }

    list, err := exampleClient.ExampleV1().Databases("default").List(metav1.ListOptions{})
    if err != nil {
        glog.Fatalf("列出所有数据库时出错: %v", err)
    }

    for _, db := range list.Items {
        fmt.Printf("数据库 %s，用户为 %q\n", db.Name, db.Spec.User)
    }
}
```

## 实战

使用 go1.21

```
brew unlink go
brew link go@1.21
```

创建 go project

```shell
mkdir code-generator-test && cd code-generator-test
go mod init code-generator-test
```

下载依赖

```shell
go get k8s.io/api@v0.29.15
go get k8s.io/apimachinery@v0.29.15
go get k8s.io/client-go@v0.29.15
go get k8s.io/code-generator@v0.29.15
```

定义crd

```shell
mkdir -p api/samplecontroller/v1alpha1
```

> 这里 group 为 samplecontroller, version 为 v1alpha1

定义`doc.go`

```go
// +k8s:deepcopy-gen=package
// +groupName=samplecontroller.k8s.io

// v1alpha1版本的api包
package v1alpha1
```

定义`types.go`

```go
package v1alpha1

import (
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
)

// +genclient
// +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object

// Foo is a specification for a Foo resource
type Foo struct {
	metav1.TypeMeta   `json:",inline"`
	metav1.ObjectMeta `json:"metadata,omitempty"`

	Spec   FooSpec   `json:"spec"`
	Status FooStatus `json:"status"`
}

// FooSpec is the spec for a Foo resource
type FooSpec struct {
	DeploymentName string `json:"deploymentName"`
	Replicas       *int32 `json:"replicas"`
}

// FooStatus is the status for a Foo resource
type FooStatus struct {
	AvailableReplicas int32 `json:"availableReplicas"`
}

// +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object

// FooList is a list of Foo resources
type FooList struct {
	metav1.TypeMeta `json:",inline"`
	metav1.ListMeta `json:"metadata"`

	Items []Foo `json:"items"`
}
```

创建目录

```shell
mkdir hack
```

创建 tools.go 来依赖 code-generator

```go
//go:build tools
// +build tools

/*
Copyright 2019 The Kubernetes Authors.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
*/

// This package imports things required by build scripts, to force `go mod` to see them as dependencies
package tools

import _ "k8s.io/code-generator"
```

编写构建脚本 update-codegen.sh

```sh
#!/usr/bin/env bash

# Copyright 2017 The Kubernetes Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

set -o errexit
set -o nounset
set -o pipefail

# generate the code with:
# --output-base    because this script should also be able to run inside the vendor dir of
#                  k8s.io/kubernetes. The output-base is needed for the generators to output into the vendor dir
#                  instead of the $GOPATH directly. For normal projects this can be dropped.
../vendor/k8s.io/code-generator/generate-groups.sh \
  "deepcopy,client,informer,lister" \
  code-generator-test/generated \
  code-generator-test/api \
  samplecontroller:v1alpha1 \
  --go-header-file $(pwd)/boilerplate.go.txt \
  --output-base $(pwd)/../../
```

创建文件头 boilerplate.go.txt

```txt
/*
Copyright The Kubernetes Authors.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
*/
```

此时目录结构为：

```
.
├── api
│   └── samplecontroller
│       └── v1alpha1
│           ├── doc.go
│           └── types.go
├── go.mod
├── go.sum
└── hack
    ├── boilerplate.go.txt
    ├── generate-groups.sh
    └── tools.go
```

接下来就可以开始生成 crd 资源的 clientset 等 api 了

生成vendor文件夹

```shell
go mod vendor
```

为 vendor 中的 code-generator 赋予权限

```shell
chmod -R 777 vendor
```

调用脚本生成代码

```shell
chmod -R 777 hack/update-codegen.sh
cd hack && ./update-codegen.sh
```

执行结果为：

```
WARNING: generate-groups.sh is deprecated.
WARNING: Please use k8s.io/code-generator/kube_codegen.sh instead.
WARNING: generate-internal-groups.sh is deprecated.
WARNING: Please use k8s.io/code-generator/kube_codegen.sh instead.
Generating deepcopy funcs
Generating clientset for samplecontroller:v1alpha1 at code-generator-test/generated/clientset
Generating listers for samplecontroller:v1alpha1 at code-generator-test/generated/listers
Generating informers for samplecontroller:v1alpha1 at code-generator-test/generated/informers
```

用 `tree -L 3` 看看此时的目录结构：

```
.
├── api
│   └── samplecontroller
│       └── v1alpha1
├── generated
│   ├── clientset
│   │   └── versioned
│   ├── informers
│   │   └── externalversions
│   └── listers
│       └── samplecontroller
├── go.mod
├── go.sum
├── hack
│   ├── boilerplate.go.txt
│   ├── tools.go
│   └── update-codegen.sh
└── vendor
    ├── github.com
    │   ├── emicklei
    │   ├── go-logr
    │   ├── go-openapi
    │   ├── gogo
    │   ├── golang
    │   ├── google
    │   ├── josharian
    │   ├── json-iterator
    │   ├── mailru
    │   ├── modern-go
    │   └── spf13
    ├── golang.org
    │   └── x
    ├── google.golang.org
    │   └── protobuf
    ├── gopkg.in
    │   ├── inf.v0
    │   ├── yaml.v2
    │   └── yaml.v3
    ├── k8s.io
    │   ├── apimachinery
    │   ├── code-generator
    │   ├── gengo
    │   ├── klog
    │   ├── kube-openapi
    │   └── utils
    ├── modules.txt
    └── sigs.k8s.io
        ├── json
        ├── structured-merge-diff
        └── yaml
```

可见，`code-generator-test/api/samplecontroller/v1alpha1`下多出了一个`zz_generated.deepcopy.go`的文件,在`generated`文件夹下生成了`clientset`和`informers`和`listers`三个文件夹

