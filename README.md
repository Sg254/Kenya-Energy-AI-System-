# Kenya Energy Access AI System - Documentation

## Model Card: Credit Scoring Model

### Model Details
- **Model Name**: Energy Access Credit Scoring Model v1.0
- **Model Type**: XGBoost Gradient Boosted Trees
- **Version**: 1.0.0
- **Date**: January 2025
- **Owner**: Kenya Energy AI Team
- **Contact**: ml-team@energy-ai.ke

### Intended Use
**Primary Use Cases:**
- Assess creditworthiness of PAYG (Pay-As-You-Go) energy customers
- Optimize payment collection strategies
- Identify customers at risk of default
- Support expansion of energy access to underserved areas

**Target Users:**
- Energy providers (utilities, mini-grids, solar companies)
- Government policy makers (REREC, KPLC)
- Financial institutions
- Development organizations

**Out-of-Scope Uses:**
- Should NOT be used as sole determinant for service denial
- Not designed for credit decisions outside energy access context
- Not intended for individual loan approval decisions

### Training Data
**Data Sources:**
- Smart meter readings (60%)
- M-PESA payment transactions (25%)
- Customer demographic data (10%)
- Grid reliability data (5%)

**Data Size:**
- Training: 500,000 customer records
- Validation: 100,000 customer records
- Time period: 2023-2024 (24 months)

**Data Preprocessing:**
- PII anonymization using SHA-256 hashing
- Location generalization to 2 decimal places
- Missing value imputation using median/mode
- Outlier capping at 99th percentile

**Feature Engineering:**
- Payment behavior features (15 features)
- Energy usage patterns (12 features)
- Demographic indicators (8 features)
- Temporal features (5 features)
- **Total Features**: 40

### Model Architecture
```
XGBoost Parameters:
- max_depth: 6
- learning_rate: 0.1
- n_estimators: 100
- min_child_weight: 3
- gamma: 0.1
- subsample: 0.8
- colsample_bytree: 0.8
- objective: binary:logistic
- eval_metric: auc
```

### Performance Metrics

**Overall Performance:**
- AUC-ROC: 0.82
- Precision: 0.76
- Recall: 0.79
- F1-Score: 0.77
- Accuracy: 0.81

**Performance by Region:**
| Region | AUC | Precision | Recall |
|--------|-----|-----------|--------|
| Urban  | 0.85| 0.79      | 0.82   |
| Rural  | 0.79| 0.73      | 0.76   |

**Performance by Demographics:**
| Group | AUC | False Positive Rate |
|-------|-----|---------------------|
| All   | 0.82| 0.18               |
| Male  | 0.83| 0.17               |
| Female| 0.81| 0.19               |

### Ethical Considerations

**Fairness Analysis:**
- Model tested for disparate impact across gender, age, location
- Max demographic parity difference: 0.08 (below 0.1 threshold)
- Equal opportunity difference: 0.06
- Regular bias audits conducted quarterly

**Privacy Protections:**
- All PII hashed before model training
- Location data generalized to prevent re-identification
- Data retention policy: 2 years maximum
- GDPR-compliant data handling

**Limitations:**
- Performance degrades for customers with <3 months history
- Less accurate for off-grid solar systems vs. mini-grids
- Requires regular retraining due to changing payment behaviors
- May not capture seasonal agricultural income patterns in rural areas

**Potential Risks:**
- Over-reliance could lead to exclusion of marginalized groups
- Model drift if payment patterns change rapidly
- False positives could unnecessarily restrict energy access
- Requires human oversight for final decisions

### Monitoring & Maintenance

**Monitoring Plan:**
- Daily: Prediction volume, API latency
- Weekly: Feature distribution drift, prediction distribution
- Monthly: Model performance metrics, bias metrics
- Quarterly: Full model retraining and evaluation

**Retraining Triggers:**
- AUC drops below 0.75
- Demographic parity exceeds 0.1
- Significant feature drift detected
- Major market changes (policy, technology)

**Model Updates:**
- Regular retraining: Monthly
- Emergency retraining: As needed
- Version control: MLflow + Git
- A/B testing for new versions

### Usage Guidelines

**Decision Thresholds:**
- Low Risk: Probability < 0.3 (Green light)
- Medium Risk: 0.3 ≤ Probability < 0.6 (Review required)
- High Risk: Probability ≥ 0.6 (Manual review + mitigation)

**Best Practices:**
1. Always review high-risk predictions manually
2. Combine with qualitative customer information
3. Provide explanations using SHAP values
4. Allow customers to dispute decisions
5. Regular fairness audits
6. Document all decisions

**Implementation Code:**
```python
from src.models.credit_scoring.model import CreditScoringModel

# Load model
model = CreditScoringModel()
model.load_model('models/credit_scoring_v1.json')

# Prepare data
features = model.prepare_features(customer_data)

# Get predictions with explanations
predictions, explanations = model.predict_with_explanation(features)

# Apply decision logic
for idx, (pred, exp) in enumerate(zip(predictions, explanations)):
    if pred < 0.3:
        action = "Approve - Low Risk"
    elif pred < 0.6:
        action = "Review - Medium Risk"
    else:
        action = "Detailed Review - High Risk"
    
    print(f"Customer {idx}: {action} (Score: {pred:.2f})")
```

---

## System Architecture Documentation

### Overview
The Kenya Energy Access AI System is a comprehensive platform designed to improve energy access through data-driven decision making. It combines smart meter data, payment information, and satellite imagery to optimize energy distribution and credit assessment.

### Architecture Layers

#### 1. Data Collection Layer
**Components:**
- Smart meter collectors (IoT devices)
- PAYG system integrations (M-PESA, Stripe)
- Satellite data ingestion (AWS Ground Station)
- Grid monitoring sensors

**Technologies:**
- MQTT for IoT messaging
- REST APIs for payment systems
- Apache Kafka for streaming data
- AWS S3 for satellite imagery

#### 2. Data Pipeline & Storage
**ETL Workflow:**
```
Extract → Validate → Transform → Anonymize → Load
```

**Technologies:**
- Apache Airflow for orchestration
- PySpark for large-scale processing
- PostgreSQL for transactional data
- AWS S3 as data lake
- Parquet for columnar storage

**Data Quality Checks:**
- Schema validation
- Null value checks
- Outlier detection
- Freshness monitoring
- Duplicate detection

#### 3. ML Models Layer
**Models:**
1. **Credit Scoring**: XGBoost classifier (AUC: 0.82)
2. **Site Selection**: Random Forest + GIS (Accuracy: 0.88)
3. **Dispatch Optimization**: DQN Reinforcement Learning
4. **Load Forecasting**: LSTM (MAPE: 8.5%)

**Model Serving:**
- FastAPI for REST endpoints
- Redis for caching
- Model versioning with MLflow
- A/B testing framework

#### 4. Application Layer
**Interfaces:**
- **Operator Dashboard**: React + D3.js
- **Government Portal**: Vue.js + Mapbox
- **Mobile App**: React Native
- **API Gateway**: Kong

#### 5. Monitoring & Governance
**Components:**
- Model performance tracking (EvidentlyAI)
- Bias detection (AIF360)
- Data lineage (Apache Atlas)
- Logging (ELK Stack)
- Alerting (PagerDuty)

### Security Architecture
- Role-based access control (RBAC)
- API authentication via OAuth 2.0
- Data encryption at rest (AES-256)
- Data encryption in transit (TLS 1.3)
- PII anonymization pipeline
- Regular security audits

### Deployment Architecture
```
Internet → Load Balancer → API Gateway
    ↓
Kubernetes Cluster
├── ML Service Pods
├── API Service Pods
├── Worker Pods (Airflow)
└── Monitoring Pods
    ↓
AWS RDS (PostgreSQL)
AWS S3 (Data Lake)
AWS ElastiCache (Redis)
```

### Scalability Considerations
- Horizontal pod autoscaling (HPA)
- Database read replicas
- CDN for static assets
- Async processing with Celery
- Rate limiting and throttling

### Disaster Recovery
- Daily database backups
- Cross-region replication
- RPO: 1 hour
- RTO: 4 hours
- Automated failover procedures

---

## Data Dictionary

### Smart Meter Data
| Field | Type | Description | Example |
|-------|------|-------------|---------|
| meter_id | string | Unique meter identifier | "MTR-2024-001234" |
| customer_id | string | Hashed customer ID | "a3f5e..." |
| timestamp | datetime | Reading timestamp | "2025-01-15 14:30:00" |
| kwh_used | float | Energy consumed (kWh) | 12.5 |
| voltage | float | Line voltage (V) | 230.2 |
| current | float | Current draw (A) | 8.3 |
| power_factor | float | Power factor | 0.92 |

### Payment Data
| Field | Type | Description | Example |
|-------|------|-------------|---------|
| payment_id | string | Unique payment ID | "PAY-2024-567890" |
| customer_id | string | Hashed customer ID | "a3f5e..." |
| amount | decimal | Payment amount (KES) | 500.00 |
| method | string | Payment method | "M-PESA" |
| timestamp | datetime | Payment time | "2025-01-15 10:15:00" |
| status | string | Payment status | "completed" |

### Customer Features
| Field | Type | Description | Range/Values |
|-------|------|-------------|--------------|
| avg_payment_amount | float | Average payment | 0-10000 KES |
| payment_frequency | int | Payments per month | 0-30 |
| days_since_last_payment | int | Days since last payment | 0-365 |
| on_time_ratio | float | On-time payment rate | 0.0-1.0 |
| avg_daily_usage | float | Average daily kWh | 0-50 |
| usage_consistency | float | Usage pattern stability | 0.0-1.0 |
| account_age_days | int | Account age | 0-3650 |
| location_type | string | Urban/Rural | "urban", "rural" |

---

## API Documentation

### Authentication
```bash
POST /api/v1/auth/token
Content-Type: application/json

{
  "username": "api_user",
  "password": "secure_password"
}

Response:
{
  "access_token": "eyJhbGc...",
  "token_type": "bearer",
  "expires_in": 3600
}
```

### Credit Scoring Endpoint
```bash
POST /api/v1/credit/score
Authorization: Bearer {token}
Content-Type: application/json

{
  "customer_id": "C12345",
  "include_explanation": true
}

Response:
{
  "customer_id": "C12345",
  "credit_score": 0.35,
  "risk_category": "Low",
  "recommendation": "Approve",
  "top_factors": [
    {"feature": "on_time_ratio", "impact": 0.12},
    {"feature": "payment_frequency", "impact": 0.08}
  ],
  "timestamp": "2025-01-15T14:30:00Z"
}
```

### Batch Scoring
```bash
POST /api/v1/credit/batch-score
Authorization: Bearer {token}
Content-Type: application/json

{
  "customer_ids": ["C001", "C002", "C003"],
  "async": true
}

Response:
{
  "job_id": "job-12345",
  "status": "processing",
  "estimated_completion": "2025-01-15T14:35:00Z"
}
```

---

## Deployment Guide

### Prerequisites
- Docker 20.10+
- Kubernetes 1.24+
- Python 3.10+
- PostgreSQL 14+
- Redis 7+

### Local Development Setup
```bash
# Clone repository
git clone https://github.com/your-org/kenya-energy-ai.git
cd kenya-energy-ai

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Set up environment variables
cp .env.example .env
# Edit .env with your configuration

# Initialize database
python scripts/init_database.py

# Run tests
pytest tests/ -v

# Start development server
python api/main.py
```

### Docker Deployment
```bash
# Build image
docker build -t kenya-energy-ai:latest -f deployment/docker/Dockerfile .

# Run container
docker run -p 8000:8000 \
  -e DB_HOST=localhost \
  -e DB_PASSWORD=secure_pass \
  kenya-energy-ai:latest
```

### Kubernetes Deployment
```bash
# Apply configurations
kubectl apply -f deployment/kubernetes/namespace.yaml
kubectl apply -f deployment/kubernetes/configmap.yaml
kubectl apply -f deployment/kubernetes/secret.yaml
kubectl apply -f deployment/kubernetes/deployment.yaml
kubectl apply -f deployment/kubernetes/service.yaml

# Check status
kubectl get pods -n energy-ai
kubectl logs -f deployment/ml-service -n energy-ai
```

---

## Troubleshooting Guide

### Common Issues

**Issue: Model predictions are inconsistent**
- Check feature drift metrics
- Verify data preprocessing pipeline
- Review recent data quality issues
- Compare with baseline model

**Issue: High API latency**
- Check Redis cache hit rate
- Review database query performance
- Monitor CPU/memory usage
- Consider horizontal scaling

**Issue: Airflow DAG failures**
- Check task logs in Airflow UI
- Verify database connections
- Review data validation errors
- Check S3 permissions

### Contact & Support
- Technical Issues: tech-support@energy-ai.ke
- Model Questions: ml-team@energy-ai.ke
- Security Concerns: security@energy-ai.ke
- General Inquiries: info@energy-ai.ke
