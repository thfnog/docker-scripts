apply:
	kubectl apply -f deployment.yaml -n local

delete:
	kubectl delete -f deployment.yaml -n local

up:
	kubectl patch flinkdeployment/basic-checkpoint-ha-example --type=merge -p '{"spec": {"job": {"state": "running"}}}' -n local

down:
	kubectl patch flinkdeployment/basic-checkpoint-ha-example --type=merge -p '{"spec": {"job": {"state": "suspended"}}}' -n local

savepoint:
	kubectl patch flinkdeployment/basic-checkpoint-ha-example --type=merge -p '{"spec": {"job": {"state": "suspended", "upgradeMode": "savepoint}}}' -n local
