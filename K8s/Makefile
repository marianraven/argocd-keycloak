base: install-nginx-ingress install-olm install-cert-manager create-namespace install-keycloak install-metrics 

install-nginx-ingress:
	helm repo add nginx-stable https://helm.nginx.com/stable
	helm repo update
	helm upgrade --install ingress-nginx ingress-nginx  --repo https://kubernetes.github.io/ingress-nginx  --namespace ingress-nginx --create-namespace

install-olm:
# importante usar create y no apply por que da error
	kubectl create -f https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.25.0/crds.yaml
	kubectl create -f https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.25.0/olm.yaml
	timeout 120
	kubectl rollout status --namespace olm deployment/olm-operator --timeout=600s

install-cert-manager:
# antes tiene que tener instalado el OLM
#Pararse en la carpeta k8s
	kubectl apply -f https://operatorhub.io/install/cert-manager.yaml
	timeout 210
	kubectl rollout status --namespace operators deployment/cert-manager --timeout=800s
	kubectl apply -f ./resources/self-sign-cluster-issuer.yaml

create-namespace:
	kubectl get namespace keycloak || kubectl create namespace keycloak
	kubectl get namespace infra || kubectl create namespace infra
	kubectl apply -f ./resources/conf.yaml -n keycloak

install-postgres: create-namespace
	kubectl apply -f ./resources/postgres/postgres.yaml -n keycloak

install-pgadmin: create-namespace
	kubectl apply -f ./resources/pgadmin/pgadmin.yaml -n keycloak

install-keycloak: create-namespace install-postgres  install-pgadmin
	kubectl rollout status --namespace  keycloak statefulset/database --timeout=600s
	helm repo add pqs https://pqsdev.github.io/helm-charts
	helm repo update pqs
	helm upgrade --install sso pqs/keycloak -f ./resources/keycloak/keycloak.yaml -n keycloak

install-metrics:
	kubectl apply -f ./resources/infra/grafana-conf.yaml -n infra
	helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
	helm repo update prometheus-community
	helm upgrade --install metrics prometheus-community/kube-prometheus-stack -f ./resources/infra/kube-prometheus-stack.yaml -n infra --create-namespace