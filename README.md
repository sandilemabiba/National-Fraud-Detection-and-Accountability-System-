This project is licensed under the GNU General Public License v3.0.
Prior to the inclusion of the LICENSE file, no license was granted for reuse, redistribution, or deployment.
NFDAS is published for academic and evaluation purposes.
Any production, commercial, or governmental deployment requires written authorization from the author.

# National-Fraud-Detection-and-Accountability-System-
A rule-based fraud detection system designed using computer science principles.

# Project Overview:
NFDAS is a conceptual fraud detection system designed to demonstrate rule-based decision-making, system constraints, and safe handling of edge cases. Detects anomalous transactions, identity theft and unauthorized data changes in real-time. Helps align with POPIA and NIST compliance standards. AI-DRIVEN fraud scoring can integrate with existing systems. This project is a conceptual system design intended to demonstrate analytical thinking, system decomposition, and problem-solving approaches in computer science.

# Problem Statement:
Large institutions continue to suffer major data breaches and procurement fraud (e.g., university systems, public hospitals). Root causes include fragmented systems, poor endpoint visibility, and the absence of unified, tamper-resistant audit trails. These weaknesses enable unauthorized access, delayed detection, and limited accountability.

# Introducing NFDAS 
1. AI-powered anomaly detection. 
2. Secure data auditing.
3. BlockChain-based traceability
4. Endpoint and response correlation

Goal: Real-time prevention and accountability. 

# Architecture Overview
CORE LAYERS:
1. Data ingestion (kafka): secure stream processing. 
2. Processing Engine: AI + ML pattern detection.
3. Storage (PostgreSQL): immutable audit logs.
4. Security Layer: TPM, anomaly monitors
5. Response Layer: Dashboard + Alerts

# NFDAS Data Flow Diagram
 ![1000105516](https://github.com/user-attachments/assets/b9ba3c31-bad6-4392-af25-94284654ee8b)
Figure 1: NFDAS secure data flow showing ingestion, validation, immutable storage, and audit layers


# Technical Features:
1. Kafka - real-time data ingestion
2. PostgreSQL - secure encrypted logging
3. ELK Stack + Prometheus Monitoring
4. TensorFlow - Pattern recognition
5. TPM Enclaves  - Secure hardware
6. Encrypted Offsite Backups - Data Recovery

These technologies are presented as representative components illustrating how the system could be implemented at scale; they are not claims of a fully deployed production system.

# Integration and Use Cases:
1. Universities - secure academic and financial data
2. Hospitals - Real-time procurement audit
3. SOEs- National treasury fraud prevention

# How NFDAS Solves the Problem?
1. Detects abnormal data access patterns
2. Continuous Vulnerability Scans
3. TPM ensures device authenticity
4. DLP prevents data exfiltration

# Results and Impact:
1. 90% reduction in fraudulent transactions - Projected impact based on comparable rule-based fraud detection systems.
2. Improved trust and accountability
3. Real-time fraud prevention capability

 # Technical Appendix:
 1. Algorithms: Isolation Forests, Bayesian Inference, Blockchain audit trails
 2. Security: AES-256 encryption,TPM attestation, Zero Trust Access

# NFDAS Code snippets 
Core implementation and detection logic are intentionally excluded from this repository to prevent misuse and protect system integrity. This repository intentionally focuses on system design and architecture. Implementation details are withheld due to security and IP considerations 

# PostgreSQL & Kafka Demonstration

A controlled, reproducible demonstration of NFDAS’s immutable audit trail, recovery logic, and forensic response is documented in docs/postgres-kafka-demo.md.
Security Notice
This demonstration is provided for academic evaluation and architectural review only.
It uses simulated data, simplified credentials, and non-production configurations.
Core fraud detection logic, scoring thresholds, and adaptive response mechanisms are intentionally omitted.

PART A — Exact commands to run the full Docker + Kafka + Postgres demo
Folder layout assumption: you unzipped the package into ~/nfdas_demo so top-level contains docker-compose.yml, api/, engine/, producer/, artifacts/, etc.

Change to the demo folder:

cd ~/nfdas_demo
1) Start Kafka + Zookeeper + Postgres with Docker Compose
docker-compose up -d
Expected output (docker-compose will print nothing to stdout in detached mode).
Check service status:

docker-compose ps
Expected output (table):

    Name                      Command               State            Ports
--------------------------------------------------------------------------------
nfdas_demo_kafka_1    /etc/confluent/docker/run ...   Up      0.0.0.0:9092->9092/tcp
nfdas_demo_zookeeper_1 /etc/confluent/docker/run ...  Up      0.0.0.0:2181->2181/tcp
nfdas_demo_postgres_1  docker-entrypoint.sh postgres  Up      0.0.0.0:5432->5432/tcp
(wait ~20–40s for broker & DB readiness)

2) Create Kafka topics (inside Kafka container)
Get container name (example):

docker ps --filter "ancestor=confluentinc/cp-kafka" --format "{{.Names}}"
Use the container name from above (assume nfdas_demo_kafka_1):

docker exec -it nfdas_demo_kafka_1 bash -lc "\
kafka-topics --bootstrap-server localhost:9092 --create --topic transactions --partitions 1 --replication-factor 1 && \
kafka-topics --bootstrap-server localhost:9092 --create --topic audit_log --partitions 1 --replication-factor 1 && \
kafka-topics --bootstrap-server localhost:9092 --create --topic alerts --partitions 1 --replication-factor 1 \
"
Expected output (one line per created topic or no output if already exists):

Created topic transactions.
Created topic audit_log.
Created topic alerts.
Verify topics:

docker exec -it nfdas_demo_kafka_1 kafka-topics --bootstrap-server localhost:9092 --list
Expected output:

__consumer_offsets
transactions
audit_log
alerts
3) Create Postgres schema (use psql in Postgres container)
Enter container (assume nfdas_demo_postgres_1):

docker exec -it nfdas_demo_postgres_1 psql -U admin -d nfdas
At psql prompt, run:

CREATE TABLE IF NOT EXISTS transactions (
  transaction_id UUID PRIMARY KEY,
  department TEXT,
  amount NUMERIC,
  user_id TEXT,
  timestamp TIMESTAMP,
  record_hash TEXT
);
CREATE TABLE IF NOT EXISTS audit_log (
  event_id UUID PRIMARY KEY,
  action TEXT,
  target_id UUID,
  user_id TEXT,
  payload_hash TEXT,
  timestamp TIMESTAMP
);
CREATE TABLE IF NOT EXISTS quarantined (
  user_id TEXT PRIMARY KEY,
  reason TEXT,
  timestamp TIMESTAMP
);
\q
Expected output (psql echoes):

CREATE TABLE
CREATE TABLE
CREATE TABLE
4) Export environment variables (host shell)
export KAFKA_BOOTSTRAP=localhost:9092
export NFDAS_DB_DSN="postgresql://admin:securepassword@localhost:5432/nfdas"
5) Start Python virtualenv and install requirements
(Do this on your host machine, in the ~/nfdas_demo directory.)

python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
Expected pip output: normal installation logs ending with Successfully installed ...

6) Start the Kafka-enabled FastAPI (producer API)
(Use the Kafka-enabled API file api/main_kafka.py included in the package.)

uvicorn api.main_kafka:app --reload --port 8000
Expected console output (uvicorn):

INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [pid]
INFO:     Started server process [pid]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
Also, startup will attempt to connect to Kafka (AIOKafkaProducer). If Kafka unreachable you'll see an exception — confirm Docker Compose and env var.

7) Start the Kafka consumer engine (engine_kafka_postgres.py)
Open a new terminal (still within venv), then:

python engine/engine_kafka_postgres.py
Expected console output:

[ENGINE] (starts, then waits while consuming Kafka)
When it consumes messages you will see lines like:

[ENGINE] Stored transaction <uuid>
[ENGINE][BACKUP] Manifest saved: <manifest_id>
...
8) Produce a legitimate transaction (from host)
Use the producer script in the package:

python producer/send_tx.py --department Finance --amount 1100000 --user user_admin_01
Expected API response printed in terminal:

200 {"status":"accepted","mode":"kafka","transaction_id":"<uuid>"}
Expected engine console output (within engine logs):

[ENGINE] Stored transaction <uuid>
[ENGINE][BACKUP] Manifest saved: <manifest_id>
Check backup manifest file (on host, inside project artifacts/):

cat artifacts/backup_manifest.json | jq .
Expected JSON excerpt:

{
  "manifest_id": "c8b4d1c0-....",
  "timestamp": "2025-10-20T14:21:16Z",
  "files": [
    {
      "transaction_id": "2f9b1a2e-....",
      "department": "Finance",
      "amount": 1100000.0,
      "user_id": "user_admin_01",
      "timestamp": "2025-10-20T14:20:50Z"
    }
  ]
}
9) Simulate malicious deletion
You have two options to create a DELETE audit message:

A) Use the provided script (simpler demo; it appends to in-memory queue, but engine in Kafka mode expects audit_log via Kafka — you can simulate by producing to Kafka manually).

Produce an audit_log message to Kafka via Kafka console-producer (inside Kafka container):

docker exec -it nfdas_demo_kafka_1 bash -lc "cat > /tmp/delete_event.json <<'EOF'
{\"event_id\":\"$(uuidgen)\",\"action\":\"DELETE\",\"target_id\":\"<uuid>\",\"user_id\":\"attacker_user_77\",\"payload_hash\":\"fakehash\",\"timestamp\":null}
EOF
kafka-console-producer --broker-list localhost:9092 --topic audit_log < /tmp/delete_event.json
"
(substitute <uuid> with the transaction id returned earlier)

Expected engine console output:

[ALERT] Missing transaction <uuid>. Attempting restore.
[RESTORE] Restored <uuid> and quarantined attacker_user_77
B) If you prefer to use provided producer/malicious_delete.py (it appends to artifacts/in_memory_queue.jsonl) — this works when engine is running in in-memory mode. For Kafka mode, prefer method A.

10) Verify Postgres audit_log and quarantined tables
Connect to Postgres and run queries (from host via psql or inside container):

From host (if psql installed):

psql "postgresql://admin:securepassword@localhost:5432/nfdas" -c "SELECT event_id, action, target_id, user_id, timestamp FROM audit_log ORDER BY timestamp DESC LIMIT 10;"
psql "postgresql://admin:securepassword@localhost:5432/nfdas" -c "SELECT * FROM quarantined;"
Expected outputs:

audit_log will include rows like CREATE (initial ingest), DELETE (attack), RESTORE (automated), with timestamps.
quarantined will include attacker_user_77 with reason like Automated quarantine for tampering <uuid>.
Example:

            event_id             | action  |           target_id            |     user_id      | timestamp
---------------------------------+---------+--------------------------------+------------------+---------------------
 6b5a9a54-...                    | RESTORE | 2f9b1a2e-...                   | NFDAS_AUTOMATION | 2025-10-20 14:22:10
 2f0c7b81-...                    | DELETE  | 2f9b1a2e-...                   | attacker_user_77 | 2025-10-20 14:22:05
 ...
11) Generate forensic report (if engine has a command or file)
The engine creates/updates backup_manifest.json and should write logs; if you want a single consolidated forensic report, run a SQL query to export audit_log + transactions to JSON, or use the engine's report script if provided.

A sample SQL to produce a forensic JSON (psql + jq on host):

psql -A -t -F',' "postgresql://admin:securepassword@localhost:5432/nfdas" -c "COPY (SELECT json_agg(t) FROM (SELECT transaction_id, department, amount, user_id, timestamp FROM transactions) t) TO STDOUT;" > transactions.json
psql -A -t -F',' "postgresql://admin:securepassword@localhost:5432/nfdas" -c "COPY (SELECT json_agg(a) FROM (SELECT event_id, action, target_id, user_id, payload_hash, timestamp FROM audit_log ORDER BY timestamp) a) TO STDOUT;" > audit_log.json
jq -n '{"transactions': (input), "audit_log': (input)}' transactions.json audit_log.json > forensic_report.json
(Adapt if needed.)

Expected: forensic_report.json containing both arrays.

12) Tear down (stop containers)
docker-compose down
PART B — Expected console outputs (detailed examples)
Below are the exact lines you should expect to see at each step (these are examples — UUIDs & timestamps will differ):

After producing a transaction with send_tx.py:
200 {"status":"accepted","mode":"kafka","transaction_id":"2f9b1a2e-7cde-4a1f-b9a4-fd8a9c1d2e3f"}
Engine logs when storing:
[ENGINE] Stored transaction 2f9b1a2e-7cde-4a1f-b9a4-fd8a9c1d2e3f
[ENGINE][BACKUP] Manifest saved: 1c2d3e4f-....
After you produce a DELETE audit log event:
[ALERT] Missing transaction 2f9b1a2e-7cde-4a1f-b9a4-fd8a9c1d2e3f. Attempting restore.
[RESTORE] Restored 2f9b1a2e-7cde-4a1f-b9a4-fd8a9c1d2e3f and quarantined attacker_user_77
psql audit_log query:
event_id                             | action  | target_id                            | user_id            | timestamp
-------------------------------------+---------+--------------------------------------+--------------------+---------------------
1c2d3e4f-...                         | CREATE  | 2f9b1a2e-...                         | user_admin_01      | 2025-10-20 14:20:50
2f0c7b81-...                         | DELETE  | 2f9b1a2e-...                         | attacker_user_77   | 2025-10-20 14:22:05
6b5a9a54-...                         | RESTORE | 2f9b1a2e-...                         | NFDAS_AUTOMATION    | 2025-10-20 14:22:10
psql quarantined:
 user_id            | reason                                    | timestamp
--------------------+-------------------------------------------+---------------------
 attacker_user_77   | Automated quarantine for tampering ...    | 2025-10-20 14:22:10
PART C — Code walkthrough (line-by-line highlights)
I’ll walk through the important files you’ll run and explain the critical functions and what to inspect.

api/main_kafka.py (FastAPI — Kafka mode)
Key parts:

KAFKA_BOOTSTRAP = os.getenv('KAFKA_BOOTSTRAP', 'localhost:9092')
reads broker address.
producer = AIOKafkaProducer(...) created in startup_event and stopped in shutdown_event.
submit_transaction(...):
Builds payload: adds transaction_id, timestamp, record_hash.
record_hash is produced via canonical_hash(payload) which ensures deterministic JSON ordering and stable SHA-256.
Sends payload to topic transactions with await producer.send_and_wait('transactions', ...).
Health endpoint returns server time.
What to watch in logs: connection exceptions on startup (means Kafka not reachable), successful producer.send_and_wait returns without exception.

engine/engine_kafka_postgres.py (consumer + logic)
Key functions:

canonical_hash(obj) — canonical JSON + SHA-256. This matches the API producer; critical for integrity comparisons.
pg_connect() — creates a psycopg2 connection using DB_DSN. If this fails, engine will error out; ensure DSN correct.
create_backup(cursor) — reads transactions rows and writes backup_manifest.json into artifacts/. This file is your offsite manifest in the demo (in production, this would be stored on an object store with immutability).
process_transactions(consumer):
Consumes both transactions and audit_log topics.
On transactions message:
Inserts row into transactions table and writes an audit CREATE row.
Calls create_backup() to keep manifest current.
On audit_log message:
Inserts audit entry.
If action == 'DELETE' and the transactions row is missing:
Engine tries to recover by reading backup_manifest.json.
If found, inserts the transaction back and writes an audit RESTORE event.
Quarantines the user_id from the DELETE event.
If not found, flags FORENSICS escalation (manual intervention).
main():
Starts AIOKafkaConsumer for topics 'transactions','audit_log', consumes messages, and calls process_transactions.


# By ST Mabiba
