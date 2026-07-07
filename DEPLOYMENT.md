# 🛡️ BioShield – Behavioral Biometric UPI Payment System

> **Secure UPI payments using keystroke dynamics, real-time risk scoring, and fallback authentication** (OTP + Face ID)

## ✨ Features

- ✅ **Keystroke Biometrics** – Continuous behavioral authentication via typing patterns
- ✅ **Real-time Risk Scoring** – Machine learning-powered fraud detection
- ✅ **Multi-factor Authentication** – OTP & Face ID fallback
- ✅ **Self-learning Profiles** – Adaptive baselines via exponential moving average
- ✅ **Dashboard & History** – Transaction logs and security events
- ✅ **Production Ready** – Docker, Redis, PostgreSQL, Nginx, HTTPS-ready

---

## 🚀 Quick Start

### Local Development (SQLite)

```bash
# 1. Clone & setup
git clone https://github.com/Yadavsandeep-stack/Bioshield-Upi
cd Bioshield-Upi

# 2. Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Run the server
export FLASK_ENV=development
python app.py
# → http://localhost:5000
```

### Docker Deployment (PostgreSQL + Redis)

```bash
# 1. Clone repository
git clone https://github.com/Yadavsandeep-stack/Bioshield-Upi
cd Bioshield-Upi

# 2. Copy environment template
cp .env.example .env

# 3. Edit .env and set SECRET_KEY and passwords

# 4. Start all services
docker-compose up -d

# 5. Initialize database
docker-compose exec app python -c "from app import db; db.create_all()"

# → http://localhost (via nginx)
# → http://localhost:5000 (direct app)
```

---

## 📖 Configuration

Create a `.env` file (copy from `.env.example`):

```env
FLASK_ENV=production
SECRET_KEY=your-super-secret-key-change-this
DATABASE_URL=postgresql://user:pass@localhost:5432/bioshield
REDIS_URL=redis://localhost:6379/0
DEEPFACE_MODEL=VGG-Face
OTP_EXPIRY_MINUTES=5
```

---

## 📚 Pages & API Reference

| URL | Method | Auth | Description |
|-----|--------|------|-------------|
| `/` | GET | ✗ | Login page with keystroke analysis |
| `/signup` | GET | ✗ | Registration form |
| `/enroll` | GET | ✓ | Keystroke enrollment (10+ samples) |
| `/test` | GET | ✓ | Biometric recognition test |
| `/payment` | GET | ✓ | UPI payment with risk engine |
| `/dashboard` | GET | ✓ | Transaction history & security events |

### API Endpoints

#### Authentication
```bash
POST /api/signup
  { "email": "user@example.com", "password": "...", "full_name": "John Doe" }

POST /api/login
  { "email": "user@example.com", "password": "...", "keystroke_data": {...} }
```

#### Biometric Enrollment
```bash
POST /api/enroll (requires token)
  { "samples": [keystroke_data_1, keystroke_data_2, ...] }

POST /api/enroll-face (requires token)
  { "face_image": "data:image/jpeg;base64,..." }
```

#### Recognition & Payment
```bash
POST /api/test (requires token)
  { "keystroke_data": {...} }
  → { "result": "Recognized|Unrecognized", "risk_score": 0.123, ... }

POST /api/payment (requires token)
  { "recipient_upi": "user@bank", "amount": 100, "keystroke_data": {...} }
  → Returns transaction ID or requires OTP/Face ID

POST /api/otp-verify (requires token)
  { "otp": "123456", "event_id": 1, "amount": 100, "recipient_upi": "..." }

POST /api/face-verify (requires token)
  { "face_image": "data:image/jpeg;base64,...", "event_id": 1, ... }
```

#### Dashboard
```bash
GET /api/risk-history (requires token)
  → { "user": {...}, "profile": {...}, "risk_events": [...], "transactions": [...] }

GET /api/health
  → { "status": "ok", "service": "BioShield", ... }
```

---

## 🔑 Risk Engine Logic

```
predict_risk(features, profile) → ALLOW | OTP_REQUIRED | BLOCK

Score = Weighted Z-score of:
  • Dwell time deviation    (25%)
  • Flight time deviation   (30%)
  • Typing speed deviation  (25%)
  • Backspace rate delta    (10%)
  • Jitter anomaly          (10%)

score < 0.35  → ALLOW (low risk)
score < 0.65  → OTP_REQUIRED (medium risk)
score ≥ 0.65  → BLOCK (high risk)
```

---

## 🏗️ Project Structure

```
bioshield/
├── app.py                      Main Flask app + all routes
├── models.py                   SQLAlchemy ORM models
├── config.py                   Configuration management
├── requirements.txt            Python dependencies
├── Dockerfile                  Docker image
├── docker-compose.yml          Full stack setup
├── nginx.conf                  Nginx reverse proxy + rate limiting
├── .env.example                Configuration template
├── .gitignore                  Git ignore file
├── README.md                   This file
├── static/
│   ├── keystroke.js            Biometric capture library (KeystrokeCapture, EnrollmentCollector)
│   └── style.css               UI design system
└── templates/
    ├── login.html              Keystroke-based login
    ├── signup.html             Account creation
    ├── enroll.html             Keystroke baseline enrollment
    ├── test.html               Biometric recognition test
    ├── payment.html            UPI payment with risk scoring
    └── dashboard.html          Security dashboard
```

---

## 🛠️ Keystroke Capture (Frontend)

### Classes

- **`KeystrokeCapture`** – Captures raw keystroke metrics (dwell, flight, jitter, etc.)
- **`EnrollmentCollector`** – Collects multiple samples for baseline creation
- **`BiometricHUD`** – Real-time stats visualization
- **`BioShieldAPI`** – HTTP client for API calls

### Usage

```javascript
// Capture keystroke telemetry
const capture = new KeystrokeCapture({ minKeys: 5 });
capture.attach(passwordInput);

// Get typed metrics
const data = capture.getData();
// {
//   dwell_times: [120, 150, 145, ...],
//   flight_times: [80, 90, 75, ...],
//   backspace_count: 2,
//   total_keys: 15,
//   duration_ms: 2500
// }

// Enroll baseline (10 samples)
const enrollment = new EnrollmentCollector(10);
enrollment.init(passwordInput, (current, total) => {
  console.log(`${current}/${total} samples`);
});
```

---

## 🔐 Production Deployment

### Prerequisites

- **Docker & Docker Compose**
- **SSL Certificate** (Let's Encrypt or self-signed)
- **PostgreSQL 16+** (or use included container)
- **Redis 7+** (or use included container)
- **Nginx** (or use included container)

### Steps

#### 1. Clone & Configure

```bash
git clone https://github.com/Yadavsandeep-stack/Bioshield-Upi
cd Bioshield-Upi
cp .env.example .env

# Edit .env with production values
nano .env
```

#### 2. Generate SSL Certificates (Optional for HTTPS)

```bash
# Using Let's Encrypt
certbot certonly --standalone -d your-domain.com

# Or use self-signed for testing
mkdir -p certs
openssl req -x509 -newkey rsa:4096 -nodes -out certs/cert.pem -keyout certs/key.pem -days 365
```

#### 3. Start Services

```bash
docker-compose up -d

# Verify all services are running
docker-compose ps

# Check logs
docker-compose logs -f app
docker-compose logs -f postgres
docker-compose logs -f redis
```

#### 4. Test the Deployment

```bash
# Health check
curl http://localhost:5000/api/health

# Or visit in browser
http://your-server-ip
```

### Scaling & Monitoring

```bash
# Scale app service (run multiple instances)
docker-compose up -d --scale app=3

# View real-time logs
docker-compose logs -f

# Stop all services
docker-compose down

# Remove volumes (WARNING: deletes data)
docker-compose down -v
```

---

## 🧪 Testing

### Manual Test Workflow

1. **Signup** → `/signup` (create account)
2. **Enroll** → `/enroll` (capture 10 keystroke samples)
3. **Test Recognition** → `/test` (test biometric accuracy)
4. **Make Payment** → `/payment` (initiate transaction with risk check)
5. **Verify OTP** → `/api/otp-verify` (if high risk)
6. **Dashboard** → `/dashboard` (view history)

### API Testing with cURL

```bash
# Signup
curl -X POST http://localhost:5000/api/signup \
  -H "Content-Type: application/json" \
  -d '{"email":"user@test.com", "password":"secure123", "full_name":"Test User"}'

# Login
curl -X POST http://localhost:5000/api/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@test.com", "password":"secure123", "keystroke_data":{...}}'

# Payment
curl -X POST http://localhost:5000/api/payment \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"recipient_upi":"john@bank", "amount":100, "keystroke_data":{...}}'
```

---

## 🚨 Production Checklist

- [x] Change `SECRET_KEY` to a strong random value
- [x] Use PostgreSQL (not SQLite) in production
- [x] Enable Redis for OTP storage (not in-memory)
- [x] Configure HTTPS/SSL certificates
- [x] Set `FLASK_ENV=production`
- [x] Enable rate limiting (nginx configured)
- [x] Add security headers (nginx configured)
- [x] Monitor logs and errors
- [x] Regular database backups
- [x] Set strong database passwords
- [ ] Deploy Face ID service (currently mocked)
- [ ] Implement real OTP gateway (currently demo)
- [ ] Add payment gateway integration
- [ ] Enable CORS for frontend domain
- [ ] Set up monitoring & alerting

---

## 📋 Important Security Notes

1. **Never commit `.env` to Git** – Use `.env.example` template
2. **Change all default passwords** – PostgreSQL, Redis, Flask SECRET_KEY
3. **Enable HTTPS in production** – Uncomment SSL block in `nginx.conf`
4. **Restrict API access** – Add IP whitelisting or API keys
5. **Face data is sensitive** – Store in encrypted volumes
6. **OTP gateway** – Replace mock implementation with real SMS service
7. **Rate limiting** – Adjust nginx limits based on your requirements

---

## 🐛 Troubleshooting

### Port 5000 Already in Use
```bash
# Change port in docker-compose.yml
ports:
  - "5001:5000"
```

### PostgreSQL Connection Failed
```bash
# Check if postgres container is running
docker-compose ps postgres

# View postgres logs
docker-compose logs postgres

# Verify DATABASE_URL in .env
```

### Redis Connection Failed
```bash
# Check redis container
docker-compose ps redis

# Test redis connection
redis-cli -h localhost ping
```

### Face ID Not Working
- Install system dependencies: `libsm6 libxext6 libxrender-dev`
- DeepFace requires face images (enroll first)
- Check DEEPFACE_MODEL setting in .env

### High False Rejection Rate
- Enroll more samples (20+ instead of 10)
- Adjust risk score thresholds in `app.py`
- Monitor z-score weights in `predict_risk()`

---

## 📞 Support & Contribution

- **Issues**: [GitHub Issues](https://github.com/Yadavsandeep-stack/Bioshield-Upi/issues)
- **Discussions**: [GitHub Discussions](https://github.com/Yadavsandeep-stack/Bioshield-Upi/discussions)
- **Pull Requests**: Contributions welcome!

---

## 📄 License

MIT License – See LICENSE file for details

---

**Built with ❤️ for secure payments**
