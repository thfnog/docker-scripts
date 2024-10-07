port-forward:
	kubectl port-forward pod/minio 9000 9090 -n minio-flink
	#open: http://localhost:9090/

create:
	kubectl create -f deployment.yaml

delete:
	kubectl delete -f deployment.yaml --selector=app=minio-namespace

update-pod:
	kubectl apply --prune -f deployment.yaml --selector=app=minio