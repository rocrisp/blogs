# Alert System for Operator-Released Pipeline: A Guide

## Introduction
The Operator-Released Pipeline Alert System is engineered to keep an eye on the **operator-released-pipeline**. Should a failure arise, the team is promptly informed via Slack. This system resides in the team's Kubernetes cluster, specifically in the pipeline-alert namespace.

### Operational Flow

1. **Trigger:** When a Pull Request (PR) is submitted and a [failure](https://github.com/redhat-openshift-ecosystem/certified-operators/pull/2684#issuecomment-1684004951) occurs, GitHub triggers a webhook to the [Pipeline Alert application](https://console-openshift-console.apps.eng.opdev.io/k8s/ns/pipeline-alerts/core~v1~Pod).
2. **Alert:** A Slack notification titled "Operator-Release-Pipeline Failure" is sent to the designated team channel.
3. **Database Logging:** A timestamp is stored in a PostgreSQL database to track the failure.
4. **Resolution:** After the issue is fixed and the pipeline runs successfully, a new entry is added to the PostgreSQL database to indicate that the pipeline is fixed.
5. **Metrics:** The time taken to fix the pipeline is calculated based on the timestamps stored in the database.

### Setup

#### Environment Setup
1. Clone the repository: **git clone https://github.com/opdev/pipeline-alerts**
2. Navigate to the directory: **cd pipeline-alerts**
3. Apply the PostgreSQL cluster configuration: **oc apply -f deploy/sla-cluster.yaml**
4. Update environment variables: Add the following exports to **setup_env.sh**.
```
cat << EOF >> setup_env.sh
# This file sets the environment variables for Pipeline-alert script
export WEBHOOKURL=http://localhost:8080
export JIRAURL="https://issues.redhat.com/secure/RapidBoard.jspa?rapidView=13817&view=detail&selectedIssue=ISVISSUEYuri-59"
export SECRET=c90e2587c693e692c9065b9a9e723a35b66ea3f4
export PG_HOST=localhost
export PG_PORT=5432
export PG_USER=sla
export PG_DBNAME=sla
EOF
```
5. Retrieve and set the PG_PASSWORD:
```
echo "export PG_PASSWORD=\"$(kubectl get secret sla-pguser-sla -o jsonpath='{.data.password}' | base64 --decode)\"" >> setup_env.sh
```
6. Make the script executable and run it:
```
chmod +x setup_env.sh
source setup_env.sh
```

#### Database Initialization
1. Copy **deploy/init.sql** to the PostgreSQL pod and execute it.
```
cat deploy/init.sql | pbcopy
```
2. Port-forward to the PostgreSQL pod: **oc port-forward podname 5432:5432**

#### Compilation and Execution
1. Build the application: **go build -o pipeline-alert cmd/main.go**
2. Make it executable: **chmod +x pipeline-alert**
3. Run the application: **./pipeline-alert**

#### Testing
1. In a new terminal, run ngrok: **ngrok http 8080**

```
Session Status                online
Account                       Rose Crisp (Plan: Free)
Version                       3.3.4
Region                        United States (us)
Latency                       27ms
Web Interface                 http://127.0.0.1:4040
Forwarding                    https://81fc-66-187-232-65.ngrok-free.app -> http://localhost:8080
```
2. Copy the **Forwarding** address and apply it to the GitHub webhook, appending /github to the endpoint. Example can be found [here](https://github.com/rocrisp/sla-test/settings/hooks)
3. Create a PR and add a label: **operator-release-pipeline/failed**

### Monitoring and SLA and the Grafana dashboard
- **Grafana Dashboard:** Metrics are displayed [here](http://grafana-sla-pipeline-alerts.apps.eng.opdev.io/?orgId=1).
- **SLA:** Currently set to 4 hours

### Payload Inspection and Database Queries

#### Watching Payload with Ngrok
1. Open a new terminal and run Ngrok: **ngrok http 8080**.
2. Observe the Ngrok Web Interface URL at **http://127.0.0.1:4040** to monitor incoming payloads.

#### Interacting with PostgreSQL Database
1. Open a new terminal and remote shell into the PostgreSQL pod: **oc rsh podname**
2. Start the PostgreSQL interactive terminal: **psql**
3. Connect to the **sla** database: **\c sla**
4. List all the tables: **\dt**
5. Retrieve all columns and rows from the table named **commentissue**: **select * from commentissue;**



