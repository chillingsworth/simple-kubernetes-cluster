SHELL := /bin/bash

$(shell touch .env)
include .env
export

AWS_CREDS_FILE=.env

#AWS variables
#Set these before running
KOPS_GROUP_NAME=kops-group-4
KOPS_USER_NAME=kops-user-4
AWS_KOPS_STATE_BUCKET=killi-ks-bucket1

#Do not change this one
export KOPS_STATE_STORE=s3://${AWS_KOPS_STATE_BUCKET}

#KOPS cluster name
#Set this before running
#It must match your Route53 domain name
export NAME=k8s.jkilli.com

#KOPS default cluster settings
#Change these if you'd like
ZONES=us-east-1a
NODE_COUNT=3
MASTER_ZONE=us-east-1a


create-kops-group:
	aws iam create-group --group-name ${KOPS_GROUP_NAME}

attach-group-permissions:
	aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess --group-name ${KOPS_GROUP_NAME}
	aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonRoute53FullAccess --group-name ${KOPS_GROUP_NAME}
	aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess --group-name ${KOPS_GROUP_NAME}
	aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/IAMFullAccess --group-name ${KOPS_GROUP_NAME}
	aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonVPCFullAccess --group-name ${KOPS_GROUP_NAME}
	aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonSQSFullAccess --group-name ${KOPS_GROUP_NAME}
	aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonEventBridgeFullAccess --group-name ${KOPS_GROUP_NAME}

create-kops-user:
	aws iam create-user --user-name ${KOPS_USER_NAME}
	aws iam add-user-to-group --user-name ${KOPS_USER_NAME} --group-name ${KOPS_GROUP_NAME}

create-kops-access-keys:
	aws iam create-access-key --user-name ${KOPS_USER_NAME} | jq -r '.[]' | \
	tee >(echo AWS_SECRET_ACCESS_KEY=$$(jq -r '.SecretAccessKey') >> $$AWS_CREDS_FILE) \
	>(echo AWS_ACCESS_KEY_ID=$$(jq -r '.AccessKeyId') >> $$AWS_CREDS_FILE) ; \
	set -o allexport && source ${AWS_CREDS_FILE} && set +o allexport

create-k8s-backend:
	aws s3api create-bucket \
	--bucket ${AWS_KOPS_STATE_BUCKET} \
	--region us-east-1

create-aws-cluster:
	kops create cluster \
	--name ${NAME} \
	--cloud aws \
	--state ${KOPS_STATE_STORE} \
	--node-count ${NODE_COUNT} \
	--zones ${ZONES}

configure-cluster:
	kops edit cluster --name ${NAME}

deploy-aws-cluster:
	kops update cluster --name ${NAME} --yes --admin

set-k8s-context:
	kops export kubecfg --name ${NAME} \
	--state ${KOPS_STATE_STORE} \
	--admin ;

create: create-kops-group attach-group-permissions create-kops-user create-kops-access-keys \
create-k8s-backend create-aws-cluster deploy-aws-cluster set-k8s-context 

validate-cluster:
	kops validate cluster --wait 10m

get-nodes:
	kubectl get nodes

health-check: validate-cluster get-nodes 

delete-aws-cluster:
	kops delete cluster \
	--name ${NAME} \
	--yes

delete-k8s-backend:
	aws s3api delete-bucket \
	--bucket ${AWS_KOPS_STATE_BUCKET} \
	--region us-east-1

delete-kops-user:
	aws iam remove-user-from-group --user-name ${KOPS_USER_NAME} --group-name ${KOPS_GROUP_NAME}
	aws iam delete-access-key --access-key-id $(shell aws iam list-access-keys --user-name ${KOPS_USER_NAME} | jq '.[][] .AccessKeyId') --user-name ${KOPS_USER_NAME}
	aws iam delete-user --user-name ${KOPS_USER_NAME}

remove-kops-group-policies:
	aws iam detach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess --group-name ${KOPS_GROUP_NAME}
	aws iam detach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonRoute53FullAccess --group-name ${KOPS_GROUP_NAME}
	aws iam detach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess --group-name ${KOPS_GROUP_NAME}
	aws iam detach-group-policy --policy-arn arn:aws:iam::aws:policy/IAMFullAccess --group-name ${KOPS_GROUP_NAME}
	aws iam detach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonVPCFullAccess --group-name ${KOPS_GROUP_NAME}
	aws iam detach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonSQSFullAccess --group-name ${KOPS_GROUP_NAME}
	aws iam detach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonEventBridgeFullAccess --group-name ${KOPS_GROUP_NAME}

delete-kops-group:
	aws iam delete-group --group-name ${KOPS_GROUP_NAME}

remove-local-env:
	rm .env

destroy: delete-aws-cluster delete-k8s-backend delete-kops-user remove-kops-group-policies delete-kops-group remove-local-env