## reddit-etl-pipeline

> This is a Tesla Energy interview project demonstrating real-time data engineering skills.

# Reddit Energy Sentiment Pipeline - Updated Rules & Context
# Last Updated: November 21, 2024
# Interview: Monday (2 days away)
# Current Status: Core pipeline complete, adding advanced analytics

## Project Context
This is a Tesla Energy interview project demonstrating real-time data engineering skills.
The pipeline processes Reddit discussions about renewable energy to extract insights,
mirroring Tesla's need to process Powerwall telemetry data.

## Current Architecture Status

### ✅ COMPLETED & WORKING (DO NOT CHANGE):
1. Docker infrastructure (5 containers: Zookeeper, Kafka, PostgreSQL, Airflow x2)
2. Reddit ingestion pipeline with Kafka producer (reddit_kafka/producer.py)
3. Kafka consumer integrated into Airflow DAG (writes to PostgreSQL)
4. Database schema with 4 tables (posts_raw, sentiment_timeseries, daily_metrics, data_quality_metrics)
5. Sentiment analysis using VADER (utils/sentiment_analyzer.py)
6. Data quality monitoring with 4 checks (monitoring/quality_checks.py)
7. Daily analytics DAG (airflow/dags/daily_analytics_dag.py)
8. 300+ posts ingested with sentiment analyzed (57% positive, 18% negative, 25% neutral)

### 🚧 IN PROGRESS (FOCUS HERE):
1. Topic extraction & issue clustering (analytics/topic_extractor.py) - PRIORITY 1
2. Real-time anomaly detection (analytics/anomaly_detector.py) - PRIORITY 2
3. Updated dashboard to show new insights - PRIORITY 3

### ⚠️ NOT IMPLEMENTED (DON'T BUILD UNLESS ASKED):
1. Spark Streaming deployment (code exists but commented out for M1 Mac performance)
2. Cloud deployment (architecture ready, but demo runs locally)
3. Advanced ML models
4. CI/CD pipeline

## Current Data State
- 300 posts in posts_raw table
- 300 sentiment records in sentiment_timeseries
- 2 daily metrics records
- 5 quality check records
- All running on localhost (Docker containers)

## Technical Decisions Made (DO NOT CHANGE):

### Database:
- PostgreSQL with direct connections (not PostgresHook due to auth issues)
- PRIMARY KEY on posts_raw.id for deduplication
- JSONB for flexible schema (details in data_quality_metrics, top_posts in daily_metrics)
- Indexes on timestamp fields for time-series queries

### Kafka:
- Topic: reddit_raw_posts
- Manual offset commits for reliability
- No partitioning (single broker, small scale)
- Consumer integrated in Airflow DAG (not separate service)

### Airflow:
- LocalExecutor (not CeleryExecutor)
- Manual triggers for all DAGs (schedule_interval=None except daily_analytics)
- Direct psycopg2 connections (not using Airflow Connections)
- Environment variables: POSTGRES_HOST, POSTGRES_DB, POSTGRES_USER, POSTGRES_PASSWORD

### Sentiment Analysis:
- VADER (not Hugging Face or custom models)
- Runs as standalone script with exported env vars
- Not integrated into streaming pipeline (manual execution)
- Processes all posts at once (batch mode)

## Known Issues & Accepted Trade-offs:

1. **Duplicate posts from /hot endpoint:** Accepted - PRIMARY KEY handles it
2. **Manual sentiment analysis:** Accepted - works for demo scale
3. **No Spark deployment:** Accepted - code ready but not needed for demo
4. **Basic HTML dashboard:** Accepted - functional but needs improvement
5. **Limited post volume (300 posts):** Accepted - Reddit API limitation for demo

## Code Style & Patterns to Follow:

### Python:
- Type hints for function parameters and returns
- Docstrings for all functions (Google style)
- Error handling with try-except and logging
- Environment variables with os.getenv() and defaults
- Use existing logger: `from utils.logger import setup_logger`

### SQL:
- Use parameterized queries (psycopg2 %s placeholders)
- Include ON CONFLICT clauses for idempotency
- Add indexes for all frequently queried fields
- Use JSONB for flexible/nested data

### Airflow DAGs:
- default_args with retries, retry_delay, execution_timeout
- Manual trigger (schedule_interval=None) unless explicitly scheduled
- Tags for categorization
- Task IDs in snake_case
- Use PythonOperator for Python code (not BashOperator with python -c)

### File Structure:
- Place analytics code in analytics/ directory
- Place monitoring code in monitoring/ directory
- Place utility code in utils/ directory
- Airflow DAGs go in airflow/dags/
- SQL migrations in sql/ directory

## Next Features to Implement:

### 1. Topic Extraction & Issue Clustering (PRIORITY 1)
**Files to Create:**
- analytics/topic_extractor.py
- airflow/dags/topic_extraction_dag.py

**New Tables:**
```sql
CREATE TABLE topics (
    id SERIAL PRIMARY KEY,
    subreddit VARCHAR(100),
    keyword VARCHAR(100),
    score FLOAT,
    post_count INTEGER,
    avg_sentiment FLOAT,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE issue_clusters (
    id SERIAL PRIMARY KEY,
    cluster_id INTEGER,
    cluster_name VARCHAR(200),
    post_ids TEXT[],
    keywords TEXT[],
    avg_sentiment FLOAT,
    post_count INTEGER,
    severity VARCHAR(20),
    created_at TIMESTAMP DEFAULT NOW()
);
```

**Implementation Requirements:**
- Use scikit-learn TfidfVectorizer for keyword extraction
- Use KMeans clustering (n_clusters=5-10)
- Process all posts from posts_raw table
- Assign severity based on negative sentiment + post count
- Store results in both tables
- Create Airflow DAG to run daily or on-demand
- Handle empty clusters gracefully

**Success Criteria:**
- Extracts top 10 keywords per subreddit
- Creates 5-10 meaningful clusters
- Assigns human-readable cluster names
- Shows in dashboard with drill-down capability

### 2. Real-Time Anomaly Detection (PRIORITY 2)
**Files to Create:**
- analytics/anomaly_detector.py
- airflow/dags/anomaly_detection_dag.py

**New Table:**
```sql
CREATE TABLE alerts (
    id SERIAL PRIMARY KEY,
    alert_type VARCHAR(50),
    severity VARCHAR(20),
    subreddit VARCHAR(100),
    message TEXT,
    metric_value FLOAT,
    threshold FLOAT,
    detected_at TIMESTAMP DEFAULT NOW(),
    resolved BOOLEAN DEFAULT FALSE
);
```

**Implementation Requirements:**
- Calculate 7-day rolling average for sentiment
- Detect deviations > 2 standard deviations
- Check volume spikes (> 3x average)
- Monitor keyword frequency changes
- Create alerts with severity (LOW, MEDIUM, HIGH, CRITICAL)
- Store in alerts table
- Airflow DAG runs hourly
- Mark alerts as resolved when back to normal

**Success Criteria:**
- Detects sentiment drops of 50%+
- Alerts on volume spikes
- Shows active alerts in dashboard
- No false positives on normal fluctuations

### 3. Dashboard Enhancement (OPTIONAL)
**Options:**
A. Streamlit dashboard (Python, 2-3 hours, recommended)
B. Next.js dashboard (TypeScript, 5-6 hours, impressive but risky)
C. Enhanced HTML with auto-refresh (1 hour, minimal)

**If Building Dashboard:**
- Show topic clusters with drill-down
- Display active alerts prominently
- Real-time metrics (auto-refresh every 30s)
- Interactive filters (date range, subreddit)
- Dark mode
- Mobile responsive

## Environment Setup:

### Required Packages (Already Installed):
- kafka-python
- psycopg2-binary
- requests
- pyyaml
- jsonschema
- vaderSentiment
- plotly

### Additional Packages Needed:
- scikit-learn (for topic extraction)
- numpy
- pandas (for data manipulation)
- scipy (for statistical tests)

### Docker Commands:
```bash
# Start all services
docker-compose up -d

# Check status
docker ps

# View logs
docker logs airflow-scheduler --tail 100

# Run Python script in container
docker exec -it airflow-scheduler python /opt/airflow/path/to/script.py

# Access database
docker exec -it postgres psql -U reddit_user -d reddit_pipeline

# Stop services
docker-compose down
```

### Environment Variables (Already Set in docker-compose.yaml):
- KAFKA_BOOTSTRAP_SERVERS=kafka:29092
- POSTGRES_HOST=postgres
- POSTGRES_PORT=5432
- POSTGRES_DB=reddit_pipeline
- POSTGRES_USER=reddit_user
- POSTGRES_PASSWORD=reddit_pass

## Interview Preparation Context:

### Demo Flow (10 minutes):
1. Show running containers (1 min)
2. Trigger ingestion DAG in Airflow UI (2 min)
3. Show topic extraction results in database (2 min)
4. Show anomaly alerts (1 min)
5. Walk through dashboard (3 min)
6. Explain cloud scaling architecture (1 min)

### Key Talking Points:
- Lambda architecture (batch + speed layers)
- Data quality by design (validation, deduplication, monitoring)
- Idempotency patterns (ON CONFLICT, unique constraints)
- Horizontal scalability (Kafka partitions, Spark workers)
- Cost optimization (batch vs streaming trade-offs)

### Questions to Prepare For:
1. "How would you handle late-arriving data?"
   → Watermarking in Spark, windowed aggregations with grace periods
2. "What if Reddit API goes down?"
   → Circuit breaker, exponential backoff, fallback to cache, alerting
3. "How do you ensure exactly-once processing?"
   → Kafka transactions, Spark checkpointing, idempotent writes (ON CONFLICT)
4. "How would you scale to millions of messages?"
   → Kafka partitioning, Spark cluster, connection pooling, batch writes
5. "What about cost optimization?"
   → Batch processing for most use cases, streaming only for real-time needs
   → Parquet for storage, data lifecycle policies, compute auto-scaling

## Cloud Architecture (For Discussion Only):

### GCP Production Deployment:
```
Google Cloud Platform
├── Cloud Run (Airflow containers, auto-scales)
├── Cloud SQL for PostgreSQL (managed, auto-backups)
├── Pub/Sub (instead of Kafka, serverless)
├── Cloud Storage (data lake, lifecycle policies)
├── Cloud Functions (lightweight processing)
├── BigQuery (analytics, ML)
├── Cloud Monitoring (alerts to Slack/PagerDuty)
└── Cloud Load Balancer (dashboard access)
```

**Cost Estimate:** $50-100/month at this scale
**Scaling:** Can handle millions of messages by adding Cloud Run instances

### Infrastructure as Code:
- Terraform for cloud resources
- GitHub Actions for CI/CD
- Helm charts for Kubernetes (if scaling beyond Cloud Run)

## Testing Strategy:

### Unit Tests (Not Implemented Yet):
- Test topic extraction with sample data
- Test anomaly detection with known spikes
- Test database writes with duplicates
- Test Airflow DAG logic

### Integration Tests:
- End-to-end pipeline test (ingestion → analysis → storage)
- Kafka producer/consumer test
- Database connection test
- Airflow DAG execution test

### Manual Testing Checklist:
- [ ] All containers running
- [ ] Airflow UI accessible
- [ ] Ingestion DAG completes successfully
- [ ] Data appears in database
- [ ] Sentiment analysis runs
- [ ] Quality checks pass
- [ ] Topic extraction finds clusters
- [ ] Anomaly detection creates alerts
- [ ] Dashboard displays data

## Documentation Requirements:

### README.md Should Include:
1. Project overview and purpose
2. Architecture diagram
3. Setup instructions (step-by-step)
4. Running the pipeline (commands)
5. Accessing dashboards (URLs)
6. Troubleshooting common issues
7. Future enhancements
8. Interview talking points

### Code Comments Should Explain:
- Why decisions were made (not just what code does)
- Trade-offs considered
- Performance implications
- Scaling considerations

## Git Workflow:

### Branches:
- main: stable, working code
- feature/topic-extraction: current work
- feature/anomaly-detection: next work

### Commit Messages:
- Use conventional commits: feat:, fix:, docs:, refactor:
- Include context: "feat: add topic extraction with TF-IDF and KMeans"
- Reference issues if tracked: "feat: add anomaly detection (#12)"

### Before Committing:
- Test locally
- Check for sensitive data (no passwords in code)
- Update documentation if needed
- Ensure Docker containers still work

## Performance Considerations:

### Database:
- Batch inserts (100-500 records at a time)
- Use connection pooling
- Create indexes before querying large datasets
- Use EXPLAIN ANALYZE for slow queries

### Kafka:
- Set appropriate batch sizes for producer
- Use compression (gzip)
- Monitor consumer lag
- Avoid processing same messages twice

### Airflow:
- Set max_active_runs=1 to prevent overlap
- Use execution_timeout to prevent hanging tasks
- Clean up old logs periodically
- Monitor task duration

## Security Best Practices:

### Current State (Local Development):
- Passwords in docker-compose.yaml (acceptable for demo)
- No encryption (acceptable for localhost)
- Default Airflow credentials (admin/admin)

### Production Requirements (Not Implemented):
- Use secrets manager (GCP Secret Manager)
- Encrypt data at rest and in transit
- Rotate credentials regularly
- Use IAM roles instead of passwords
- Enable audit logging

## Monitoring & Alerting (If Time Permits):

### Metrics to Track:
- Pipeline latency (ingestion to storage)
- Message throughput (messages/second)
- Error rate (failed tasks / total tasks)
- Data freshness (time since last post)
- Database size growth
- Query performance

### Alerts to Configure:
- Pipeline failure (DAG fails)
- Data staleness (no new posts in 1 hour)
- Quality check failures
- Sentiment anomalies detected
- Database connection errors

## Final Checklist Before Interview:

### Technical:
- [ ] All services running
- [ ] Can start/stop entire stack with single command
- [ ] All DAGs visible in Airflow UI
- [ ] Database has current data
- [ ] Dashboard displays correctly
- [ ] Topic extraction shows meaningful results
- [ ] Anomaly detection has example alerts

### Documentation:
- [ ] README.md complete
- [ ] Architecture diagram created
- [ ] Code comments added
- [ ] Demo script written

### Preparation:
- [ ] 10-minute demo rehearsed
- [ ] Talking points memorized
- [ ] Questions anticipated and answered
- [ ] Laptop charged
- [ ] Internet backup plan (hotspot)

## AI Assistant Guidelines:

When helping with this project:

1. **Preserve existing working code** - Don't refactor what works
2. **Follow established patterns** - Use same style as existing code
3. **Test incrementally** - Build one feature at a time
4. **Document decisions** - Explain why, not just what
5. **Consider interview context** - Prioritize demo-able features
6. **Stay practical** - Don't over-engineer for demo scale
7. **Focus on insights** - Build features that answer real questions
8. **Keep it running** - Don't break existing functionality

### When User Asks for Help:
- First check what's already working
- Suggest building on existing code
- Provide complete, runnable code (not snippets)
- Include test commands
- Explain trade-offs

### When User Reports Issues:
- Ask for specific error messages
- Check logs (Airflow, container, database)
- Verify environment variables
- Test database connections
- Check Docker container status

## Success Metrics:

### Technical Success:
- Pipeline processes 300+ posts
- Zero data quality failures
- Sentiment analysis completes
- Topics extracted and clustered
- Anomalies detected and alerted
- Dashboard displays all insights

### Interview Success:
- Can demo end-to-end in 10 minutes
- Can explain architecture clearly
- Can discuss scaling to production
- Can answer trade-off questions
- Shows production-ready thinking

---

**Remember:** This is an interview project, not production system.
Focus on demonstrating skills, not building perfect software.
Quality over quantity. Working features over half-built complexity.

**Priority Order:**
1. Keep existing features working
2. Add topic extraction (highest impact)
3. Add anomaly detection (production signal)
4. Improve dashboard (nice to have)
5. Everything else (bonus)

**Time Management:**
- Saturday: Topic extraction (4 hours max)
- Sunday AM: Anomaly detection (3 hours max)
- Sunday PM: Dashboard + testing (3 hours max)
- Monday AM: Final polish + practice (2 hours max)

Good luck! 🚀

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/meshivanshsinghh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
