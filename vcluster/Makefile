GCP_PROJECT ?= cloud-native-experience-lab
GCP_REGION ?= europe-west1
GCP_ZONE ?= europe-west1-b

GITHUB_USER ?= lreimer
CLUSTER_NAME ?= capi-mgmt-vcluster
VCLUSTER_NAME ?= tenant-00

prepare-gcp:
	@gcloud config set project $(GCP_PROJECT)
	@gcloud config set compute/zone $(GCP_ZONE)
	@gcloud config set container/use_client_certificate False

create-capi-cluster:
	@gcloud container clusters create $(CLUSTER_NAME) \
	  	--release-channel=regular \
		--cluster-version=1.26 \
  		--region=$(GCP_REGION) \
		--addons=HttpLoadBalancing,HorizontalPodAutoscaling \
		--workload-pool=$(GCP_PROJECT).svc.id.goog \
		--enable-autoscaling \
		--autoscaling-profile=optimize-utilization \
		--num-nodes=1 \
		--min-nodes=1 --max-nodes=10 \
		--machine-type=e2-standard-8 \
		--logging=SYSTEM \
    	--monitoring=SYSTEM
	@kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=$$(gcloud config get-value core/account)
	@kubectl cluster-info

bootstrap-capi-flux2:
	@flux bootstrap github \
		--owner=$(GITHUB_USER) \
        --repository=k8s-fleet-capi-gitops \
        --branch=main \
        --path=./clusters/gcp/$(CLUSTER_NAME) \
		--components-extra=image-reflector-controller,image-automation-controller \
		--read-write-key \
		--personal

bootstrap-capi-cluster:
	# required to avoid API rate limiting
	@echo "You may need to set a personal GITHUB_TOKEN to avoid API rate limiting"
	@clusterctl init --infrastructure vcluster

bootstrap-vcluster-flux2:
	@flux bootstrap github \
		--owner=$(GITHUB_USER) \
        --repository=k8s-fleet-capi-gitops \
        --branch=main \
        --path=./clusters/gcp/$(CLUSTER_NAME)/$(VCLUSTER_NAME) \
		--components-extra=image-reflector-controller,image-automation-controller \
		--read-write-key \
		--personal

delete-capi-cluster:
	@gcloud container clusters delete $(CLUSTER_NAME) --region=$(GCP_REGION) --async --quiet
