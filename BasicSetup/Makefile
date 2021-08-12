
# Handy
OKBLUE := '\033[94m'
OKCYAN := '\033[96m'
OKGREEN := '\033[92m'
WARNING := '\033[93m'
FAIL := '\033[91m'
ENDC := '\033[0m'
BOLD := '\033[1m'

#===========================================


.DEFAULT: all
.PHONY: all clean kind $(SECRETS)


##########################################
###   ____                     _
###  / ___|  ___  ___ _ __ ___| |_ ___
###  \___ \ / _ \/ __| '__/ _ \ __/ __|
###   ___) |  __/ (__| | |  __/ |_\__ \
###  |____/ \___|\___|_|  \___|\__|___/
###
###

### Local git server,
### For private ArgoCD in kind
gitserver:
	docker build . -t gitserver:latest -f kind/gitserver.Dockerfile
	kind load docker-image gitserver:latest --name $(KIND_NAME)

	kubectl create namespace git || true
	kubectl apply -f kind/gitserver/Deployment.yaml
	kubectl apply -f kind/gitserver/Service.yaml
	kubectl rollout restart deployment -n git gitserver

	# Give a little grace period before going to the next steps
	sleep 30


deploy-argocd:

	kubectl create namespace argocd
	kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
	@while ! kubectl get secrets \
		-n argocd | grep -q argocd-initial-admin-secret; do \
		echo "Waiting for ArgoCD to start..."; \
		sleep 5; \
	done

	$(MAKE) argo-get-pass

argo-get-pass:
	@printf $(OKGREEN)
	@printf $(BOLD)
	@echo "ArgoCD Login"
	@echo "=========================="
	@echo "ArgoCD Username is: admin"
	@printf "ArgoCD Password is: %s\n" $$(kubectl -n argocd \
		get secret argocd-initial-admin-secret \
		-o jsonpath="{.data.password}" | base64 -d)
	@echo "=========================="
	@printf $(ENDC)

port-forward:
	kubectl port-forward -n argocd svc/argocd-server 8080:8080

seldon-install:
	kubectl create namespace seldon-system
	helm install seldon-core seldon-core-operator \
    		--repo https://storage.googleapis.com/seldon-charts \
    		--set usageMetrics.enabled=true \
    		--namespace seldon-system

