operator:
	-kubectl create -f https://github.com/jetstack/cert-manager/releases/download/v1.8.2/cert-manager.yaml
	-helm repo add flink-operator-repo https://downloads.apache.org/flink/flink-kubernetes-operator-1.8.0/
	helm install flink-kubernetes-operator flink-operator-repo/flink-kubernetes-operator --set webhook.create=false

rm-operator:
	-kubectl delete -f https://github.com/jetstack/cert-manager/releases/download/v1.8.2/cert-manager.yaml
	-helm repo remove flink-operator-repo
	-helm uninstall flink-kubernetes-operator

postgres:
	@echo "Creating postgres..."
	helm install my-release oci://registry-1.docker.io/bitnamicharts/postgresql
	@$(MAKE) port-forward-postgres

port-forward-postgres:
	@echo "Retrieving postgres password..."
	$(eval POSTGRES_PASSWORD := $(shell kubectl get secret --namespace default postgresql-release -o jsonpath="{.data.postgres-password}" | base64 -d))
	@echo "The postgres password is $(POSTGRES_PASSWORD)"
	@echo "Setting up port forwarding..."
	@kubectl port-forward --namespace default svc/postgresql-release 5432:5432 > port-forward.log 2>&1 &
	@echo "Waiting for port-forwarding to be established..."
	@sleep 5
	$(MAKE) liquibase POSTGRES_PASSWORD=$(POSTGRES_PASSWORD)

liquibase:
	@echo "Running Liquibase with password: $(POSTGRES_PASSWORD)"
	-@cd ../../ && mvn resources:resources liquibase:update -Dliquibase.password=$(POSTGRES_PASSWORD)

kill_postgres:
	@echo "Killing processes using port 5432..."
	@for pid in $$(lsof -t -i :5432); do \
		echo "Killing process $$pid"; \
		kill -9 $$pid; \
	done
	@$(MAKE) check_pid

check_pid:
    @pids=$$(lsof -t -i :5432 2>/dev/null); \
    if [ -z "$$pids" ]; then \
        echo "All pid for port 5432 was killed."; \
    else \
        echo "PIDs for port 5432: $$pids"; \
    fi

broker:
	cd kafka; kubectl apply -f .

schema:
	cd schema-registry; kubectl apply -f .

zoo:
	cd zookeeper; kubectl apply -f .

rm-broker:
	cd kafka; kubectl delete -f .

rm-schema:
	cd schema-registry; kubectl delete -f .

rm-zoo:
	cd zookeeper; kubectl delete -f .

image:
	cd ../../; docker build -t cepplatform:latest .

rm-flink:
	-cd flink; kubectl delete -f deployment.yaml

pvc:
	-cd flink/pvc; kubectl create -f checkpoint-pv.yaml
	-cd flink/pvc; kubectl create -f checkpoint-pvc.yaml

config:
	cd flink/configs; kubectl apply -f config-map.yaml
	cd flink/configs; kubectl apply -f job-config-map.yaml

update-flink: rm-flink
	cd flink; kubectl apply -f deployment.yaml

kafka: zoo broker schema
rm-kafka: rm-zoo rm-broker rm-schema
deploy-flink: image pvc config flink flink

.PHONY: operator postgres zoo broker schema