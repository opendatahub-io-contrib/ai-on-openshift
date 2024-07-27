BASE:=$(shell dirname $(realpath $(lastword $(MAKEFILE_LIST))))
SHELL=/bin/sh

.PHONY: teardown-kueue-premption setup-kueue-premption

teardown-kueue-premption:
	
	-oc delete -f $(BASE)/team-a-cq.yaml -f $(BASE)/team-b-cq.yaml -f $(BASE)/shared-cq.yaml
	-oc delete -f $(BASE)/team-a-local-queue.yaml -f $(BASE)/team-b-local-queue.yaml	
	-oc delete -f $(BASE)/default-flavor.yaml -f $(BASE)/gpu-flavor.yaml
	-oc delete -f $(BASE)/team-a-rb.yaml -f $(BASE)/team-b-rb.yaml
	-oc delete -f $(BASE)/team-a-ns.yaml -f $(BASE)/team-b-ns.yaml
	
	@echo "Deleting all clusterqueues"
	oc delete clusterqueue --all --all-namespaces 

	@echo "Deleting all resourceflavors"
	oc delete resourceflavor --all --all-namespaces 
	
setup-kueue-premption:
	
	oc create -f $(BASE)/team-a-ns.yaml -f $(BASE)/team-b-ns.yaml
	oc create -f $(BASE)/team-a-rb.yaml -f $(BASE)/team-b-rb.yaml
	oc create -f $(BASE)/default-flavor.yaml -f $(BASE)/gpu-flavor.yaml
	oc create -f $(BASE)/team-a-cq.yaml -f $(BASE)/team-b-cq.yaml -f $(BASE)/shared-cq.yaml
	oc create -f $(BASE)/team-a-local-queue.yaml -f $(BASE)/team-b-local-queue.yaml