# 📜 Bulk Certificate Generator

A backend REST API built with **Python and Flask** that allows you to generate personalized PDF certificates for multiple recipients in bulk.

The application accepts certificate details and a list of recipients, generates an individual PDF certificate for each valid recipient, tracks the processing status, handles individual failures without stopping the entire job, and provides options to download certificates individually or as a ZIP file.

---

## 🚀 Features

* Generate certificates for multiple recipients in a single request
* Personalized certificate for every recipient
* PDF generation using **ReportLab**
* REST API built with **Flask**
* Background certificate processing using `ThreadPoolExecutor`
* Real-time job progress tracking
* Individual recipient validation
* Duplicate email detection
* Invalid recipients do not stop valid certificates from being generated
* Individual certificate download
* Download all successful certificates as a ZIP
* SQLite database for job and certificate tracking
* Atomic PDF file generation
* Configurable synchronous mode for testing
* Automated API and generation tests

---

## 🛠️ Tech Stack

| Technology         | Purpose                    |
| ------------------ | -------------------------- |
| Python 3.10+       | Backend programming        |
| Flask              | REST API                   |
| SQLite             | Database                   |
| ReportLab          | PDF certificate generation |
| ThreadPoolExecutor | Background processing      |
| Pytest / unittest  | Testing                    |
| ZIP                | Bulk certificate download  |

---

## 📁 Project Structure

```text
bulkcert/
│
├── app/
│   ├── __init__.py       # Flask application setup
│   ├── routes.py         # API endpoints
│   ├── worker.py         # Background job processing
│   ├── certgen.py        # PDF certificate generation
│   ├── validation.py     # Request and recipient validation
│   └── db.py             # SQLite database and schema
│
├── tests/
│   ├── __init__.py
│   └── test_api.py       # API and functionality tests
│
├── requirements.txt
├── run.py                # Application entry point
├── README.md
└── .gitignore
```

---

## ⚙️ Installation

### 1. Clone the repository

```bash
git clone https://github.com/your-username/bulk-certificate-generator.git
cd bulk-certificate-generator
```

### 2. Create a virtual environment

#### Windows

```bash
python -m venv .venv
.venv\Scripts\activate
```

#### Linux / macOS

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

---

## ▶️ Run the Application

Start the Flask server:

```bash
python run.py
```

The API will be available at:

```text
http://127.0.0.1:5000
```

The application automatically creates the required `data/` directory for the SQLite database and generated certificates.

---

# 🔌 API Documentation

## 1. Create a Certificate Generation Job

### Endpoint

```http
POST /jobs
```

### Request

```json
{
  "certificate": {
    "title": "Python Bootcamp",
    "issued_by": "Acme Academy",
    "issue_date": "2026-10-01",
    "description": "Completed the 4-week intensive course."
  },
  "recipients": [
    {
      "name": "Asha Rao",
      "email": "asha@example.com"
    },
    {
      "name": "Ben Carter",
      "email": "ben@example.com"
    }
  ]
}
```

### Response

```json
{
  "id": "job-id",
  "status_url": "/jobs/job-id"
}
```

HTTP Status:

```text
202 Accepted
```

---

## 2. Check Job Status

### Endpoint

```http
GET /jobs/<job_id>
```

Returns information such as:

* Job status
* Total recipients
* Successfully generated certificates
* Failed certificates
* Pending certificates
* Progress percentage
* Failure details

Example:

```json
{
  "status": "completed",
  "total": 2,
  "succeeded": 2,
  "failed": 0,
  "pending": 0,
  "progress_percent": 100.0
}
```

### Possible Job Statuses

```text
pending
processing
completed
completed_with_errors
failed
```

---

## 3. List Certificates

### Endpoint

```http
GET /jobs/<job_id>/certificates
```

You can also filter certificates by status:

```http
GET /jobs/<job_id>/certificates?status=success
```

Available statuses:

```text
success
failed
pending
```

---

## 4. Download an Individual Certificate

### Endpoint

```http
GET /jobs/<job_id>/certificates/<certificate_id>/download
```

Returns the generated PDF certificate.

---

## 5. Download All Successful Certificates

### Endpoint

```http
GET /jobs/<job_id>/download
```

Returns a ZIP file containing all successfully generated certificates.

---

# 🧪 Validation

The application performs two levels of validation.

### Request-level validation

Invalid requests are rejected with HTTP `400`.

Examples:

* Missing certificate title
* Missing issuer
* Invalid issue date
* Empty recipient list
* More than the allowed number of recipients
* Invalid request body

### Recipient-level validation

An individual invalid recipient does not stop the entire job.

For example:

```json
{
  "name": "",
  "email": "invalid-email"
}
```

will be marked as failed while valid recipients continue to receive certificates.

This provides **failure isolation** during bulk processing.

---

# 📄 Certificate Generation

Certificates are generated using **ReportLab**.

The generated certificate contains:

* Recipient name
* Certificate title
* Description
* Issue date
* Issuing organization

The application uses a predefined A4 landscape certificate template.

Long names and titles are automatically resized to fit the certificate, while descriptions are wrapped across multiple lines.

---

# ⚡ Background Processing

Certificate generation runs in the background using Python's:

```python
ThreadPoolExecutor
```

When a client submits a job, the API immediately returns:

```text
202 Accepted
```

Instead of keeping the HTTP request open until every certificate is generated.

The client can then poll:

```http
GET /jobs/<job_id>
```

to monitor progress.

For example:

```text
Pending → Processing → Completed
```

or:

```text
Pending → Processing → Completed with Errors
```

---

# 🗄️ Database

The project uses **SQLite**.

Two primary tables are used:

### `jobs`

Stores:

* Job ID
* Certificate information
* Job status
* Total recipients
* Creation time
* Completion time

### `certificates`

Stores:

* Certificate ID
* Job ID
* Recipient name
* Recipient email
* Generation status
* Error information
* Generated file path

SQLite WAL mode is enabled to improve concurrent database access.

---

# 🧪 Running Tests

Run the test suite using:

```bash
python -m pytest -v
```

Or using Python's built-in unittest:

```bash
python -m unittest discover -s tests -v
```

The tests cover:

* API request validation
* Certificate generation
* Invalid recipients
* Duplicate emails
* Job status
* Progress tracking
* Individual certificate downloads
* ZIP downloads
* Failure isolation
* Background processing
* Invalid job/certificate IDs

---

# 🔐 Error Handling

The application is designed so that one failed certificate does not stop the remaining certificates.

For example:

```text
Recipient 1 → Success
Recipient 2 → Success
Recipient 3 → Failed
Recipient 4 → Success
```

The final job status becomes:

```text
completed_with_errors
```

The API also records the reason for each failure.

---

# 📦 Storage

Generated certificates are stored under:

```text
data/certificates/<job_id>/
```

Each certificate receives a server-generated unique ID.

PDFs are first written to a temporary file and then atomically renamed to prevent partially generated files from being served.

---

# 🔮 Future Improvements

Possible improvements for a production version include:

* User authentication and authorization
* PostgreSQL instead of SQLite
* Celery/RQ for durable background jobs
* Redis message broker
* Job persistence and automatic recovery
* Pagination for large certificate lists
* Rate limiting
* Custom certificate templates
* Uploadable certificate templates
* Unicode font support for Hindi and other scripts
* Email certificates directly to recipients
* Admin dashboard
* Docker support
* Cloud storage such as AWS S3

---

# ⚠️ Current Limitations

* Background jobs use an in-process thread pool.
* Jobs are not automatically resumed if the server shuts down.
* The default PDF fonts have limited Unicode support.
* Authentication is not currently implemented.
* Certificate listing does not currently use pagination.

---

# 💡 Example Use Cases

This project can be used by:

* Educational institutes
* Online course platforms
* Training centers
* Hackathons
* Workshops
* Bootcamps
* Corporate training programs
* Event organizers
* Certification platforms

For example, an institute can submit 1,000 participants through a single API request and generate individual certificates automatically.

---

# 👨‍💻 Author

Developed as a backend/API project demonstrating:

* REST API development
* Flask
* Database design
* Background processing
* PDF generation
* Input validation
* Error handling
* Automated testing

---

## ⭐ If you find this project useful

Give the repository a ⭐ and feel free to contribute improvements.
