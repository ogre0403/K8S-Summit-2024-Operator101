CRD_NAME := myweb
VERSION := v1
BASE_PATH := web-crd
K8S_API_VER := v0.35.5

install-client-gen:
	go install k8s.io/code-generator/cmd/client-gen@$(K8S_API_VER)

install-deepcopy-gen:
	go install k8s.io/code-generator/cmd/deepcopy-gen@$(K8S_API_VER)

install-register-gen:
	go install k8s.io/code-generator/cmd/register-gen@$(K8S_API_VER)

install-informer-gen:
	go install k8s.io/code-generator/cmd/informer-gen@$(K8S_API_VER)

install-lister-gen:
	go install k8s.io/code-generator/cmd/lister-gen@$(K8S_API_VER)


install-generator: install-client-gen install-deepcopy-gen install-register-gen install-informer-gen install-lister-gen

generate-deepcopy: install-deepcopy-gen
	deepcopy-gen \
	--output-file zz_generated.deepcopy.go \
	--go-header-file ./hack/boilerplate.go.txt \
	$(BASE_PATH)/pkg/apis/$(CRD_NAME)/$(VERSION)

generate-clientset: install-client-gen
	client-gen \
	--clientset-name clientset \
	--input-base "" \
	--input $(BASE_PATH)/pkg/apis/$(CRD_NAME)/$(VERSION) \
	--output-dir pkg/ \
	--output-pkg $(BASE_PATH)/pkg/ \
	--go-header-file ./hack/boilerplate.go.txt


generate-register: install-register-gen
	register-gen \
	--output-file zz_generated.register.go \
	--go-header-file ./hack/boilerplate.go.txt \
	${BASE_PATH}/pkg/apis/${CRD_NAME}/${VERSION}

generate-informer: install-informer-gen
	informer-gen \
	--versioned-clientset-package $(BASE_PATH)/pkg/clientset \
	--listers-package $(BASE_PATH)/pkg/listers \
	--output-dir pkg/informers \
	--output-pkg $(BASE_PATH)/pkg/informers \
	--go-header-file ./hack/boilerplate.go.txt \
	$(BASE_PATH)/pkg/apis/$(CRD_NAME)/$(VERSION)

generate-lister: install-lister-gen
	lister-gen \
	--output-dir pkg/listers \
	--output-pkg $(BASE_PATH)/pkg/listers \
	--go-header-file ./hack/boilerplate.go.txt \
	$(BASE_PATH)/pkg/apis/$(CRD_NAME)/$(VERSION)

auto-generate: generate-deepcopy generate-register generate-clientset generate-informer generate-lister


remove-generated:
	rm -rf pkg/clientset
	rm -rf pkg/informers
	rm -rf pkg/listers
	rm -rf pkg/apis/$(CRD_NAME)/$(VERSION)/zz_generated.deepcopy.go
	rm -rf pkg/apis/$(CRD_NAME)/$(VERSION)/zz_generated.register.go

clean: remove-generated
	rm -rf pkg/
	rm -rf hack/
	rm main.go