# Cloud Enabled Deployment In Action with GCP

Name : Y M K J S Gunasekara

Student Id : 2301671001

Email Address : kusalgunasekara2002@gmail.com

### Video Recording
https://drive.google.com/file/d/1P1lnrguO-KBCDM5VoNvmHWpsQx86Z9M9/view?usp=sharing

This repository contains four projects:

- course-service (Spring Boot + MySQL)
- student-service (Spring Boot + MongoDB)
- media-service (Spring Boot + Local file storage, can be extended to S3/MinIO)
- frontend-app (React + TypeScript)

## Backend Services

### 1. course-service
- Entity: Course(id, name, duration)
- Endpoints:
  - GET /courses
  - GET /courses/{id}
  - POST /courses
  - DELETE /courses/{id}
- Default port: 8081
- Configure MySQL settings

### 2. student-service
- Document: Student(registrationNumber, fullName, address, contact, email)
- Endpoints:
  - GET /students
  - GET /students/{id}
  - POST /students
  - DELETE /students/{id}
- Default port: 8082
- Configure MongoDB settings

### 3. media-service
- Resource: files
- Endpoints:
  - POST /files (multipart/form-data: file)
  - GET /files (list)
  - GET /files/{id} (fetch)
  - DELETE /files/{id} (delete)
- Default port: 8083
- Uses local disk storage at `./data/media` by default (override with env var `MEDIA_STORAGE_DIR`)
- Supports Google Cloud Storage when `gcp` profile is active

## Frontend (frontend-app)
- React + TypeScript + MUI + Axios + Vite app with 3 sections: Courses, Students, Media
- Scripts:
  - npm run dev (Vite dev server)
  - npm run build (TypeScript build + Vite build)
  - npm run preview (Preview built app)

## Build

- Backend: run `mvn -q -e -DskipTests package` at repo root to build services.
- Frontend: run `npm install` then `npm run dev` inside `frontend-app`.

## Google Cloud Platform (GCP) Deployment

### GCP Project Setup

**Project ID:** `eca-cloud-deployment-project`
**Region:** `us-central1` (Iowa)

### 1. Course Service - Google Cloud SQL (MySQL) Configuration

#### Cloud SQL Instance Specifications:
- **vCPUs:** 1
- **Memory:** 3.75 GB
- **SSD Storage:** 10 GB
- **Database Version:** MySQL 8.0.41
- **Auto Storage Increase:** Enabled
- **Location:** us-central1-c
- **Port:** 3306 (default MySQL port)

#### Why These Specifications?
- **1 vCPU + 3.75GB RAM:** Sufficient for development/testing workloads. MySQL requires adequate memory for query processing and caching.
- **10GB SSD:** Provides fast I/O operations for database transactions. SSDs offer better performance than HDDs for database workloads.
- **Auto Storage Increase:** Prevents service interruption when storage fills up, automatically scaling as data grows.
- **us-central1-c:** Chosen for low latency and cost optimization in the US region.

#### Application Configuration (`application-gcp.properties`):
```properties
spring.datasource.host=google
spring.datasource.project-id=eca-cloud-deployment-project
spring.datasource.region=us-central1
spring.datasource.instance-name=course-db-instance
spring.datasource.database=eca_courses

spring.datasource.cloud-sql-instance=${spring.datasource.project-id}:${spring.datasource.region}:${spring.datasource.instance-name}

spring.datasource.url=jdbc:mysql://${spring.datasource.host}/${spring.datasource.database}?cloudSqlInstance=${spring.datasource.cloud-sql-instance}&socketFactory=com.google.cloud.sql.mysql.SocketFactory&useSSL=false

spring.datasource.username=<your-mysql-username>
spring.datasource.password=<your-mysql-password>
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

spring.jpa.database-platform=org.hibernate.dialect.MySQL8Dialect
```

#### Configuration Explanation:
- **`spring.datasource.host=google`:** Tells Spring Boot to use Google Cloud SQL proxy for secure connection
- **`cloud-sql-instance`:** Full instance connection string format: `project-id:region:instance-name`
- **`socketFactory=com.google.cloud.sql.mysql.SocketFactory`:** Enables secure connection through Cloud SQL proxy without exposing database to public internet
- **`useSSL=false`:** Cloud SQL proxy handles encryption, so direct SSL is not needed
- **`MySQL8Dialect`:** Hibernate dialect specifically for MySQL 8.0 features and syntax

### 2. Student Service - MongoDB Atlas Configuration

#### MongoDB Atlas Cluster Specifications:
- **Region:** Iowa (us-central1)
- **Version:** 8.0.13
- **Tier:** Free tier cluster
- **Cluster Name:** ECAStudentCluster

#### Why These Specifications?
- **Free Tier:** Cost-effective for development and learning purposes
- **Iowa Region:** Consistent with other services for low latency and data locality
- **MongoDB 8.0.13:** Latest stable version with improved performance and security features

#### Application Configuration (`application-gcp.properties`):
```properties
spring.data.mongodb.host=<your-cluster-name>.kmci8c0.mongodb.net
spring.data.mongodb.username=<your-mongodb-username>
spring.data.mongodb.password=<your-mongodb-password>
spring.data.mongodb.database=eca
spring.data.mongodb.port=27017

spring.data.mongodb.app-name=ECAStudentCluster

spring.data.mongodb.uri=mongodb+srv://${spring.data.mongodb.username}:${spring.data.mongodb.password}@${spring.data.mongodb.host}/${spring.data.mongodb.database}?appName=${spring.data.mongodb.app-name}
```

#### Configuration Explanation:
- **`spring.data.mongodb.host`:** MongoDB Atlas cluster endpoint. The `.kmci8c0.mongodb.net` is Atlas's domain for secure connections
- **`spring.data.mongodb.port=27017`:** Standard MongoDB port, though Atlas uses SRV records for connection discovery
- **`spring.data.mongodb.app-name`:** Identifies the application in MongoDB Atlas monitoring and logs
- **`mongodb+srv://`:** SRV connection string format that automatically handles replica set discovery and load balancing
- **`appName` parameter:** Helps with connection monitoring and debugging in Atlas dashboard

### 3. Media Service - Google Cloud Storage Configuration

#### Google Cloud Storage Bucket Specifications:
- **Bucket Name:** `eca-cloud-deployment-media-bucket` (must be globally unique)
- **Location Type:** Regional
- **Region:** `us-central1` (Iowa)
- **Storage Class:** Standard (for frequently accessed files)
- **Access Control:** Uniform (recommended)
- **Public Access:** Enabled for object viewing

#### Why These Specifications?
- **Regional Storage:** Better performance and lower latency for applications in the same region
- **Standard Storage Class:** Optimized for frequently accessed files with high availability
- **Uniform Access Control:** Simplified permission management across all objects
- **Public Access:** Allows direct access to media files via URLs without authentication

#### Application Configuration (`application-gcp.properties`):
```properties
# Google Cloud Storage Configuration
spring.cloud.gcp.storage.bucket-name=eca-cloud-deployment-media-bucket
spring.cloud.gcp.storage.project-id=eca-cloud-deployment-project

# Service Account Authentication
spring.cloud.gcp.credentials.location=/path/to/your/service-account-key.json

# Storage Configuration
spring.cloud.gcp.storage.auto-create-bucket=false
```

#### Configuration Explanation:
- **`spring.cloud.gcp.storage.bucket-name`:** Name of the GCS bucket for storing media files
- **`spring.cloud.gcp.storage.project-id`:** GCP project ID where the bucket is located
- **`spring.cloud.gcp.credentials.location`:** Path to the service account JSON key file for authentication
- **`spring.cloud.gcp.storage.auto-create-bucket=false`:** Prevents automatic bucket creation (bucket must exist beforehand)

#### Service Account Setup:
1. **Create Service Account:**
   - Go to IAM & Admin → Service Accounts
   - Create service account: `media-service-account`
   - Assign roles: `Storage Object Admin` and `Storage Object Viewer`

2. **Download Service Account Key:**
   - Create and download JSON key file
   - Store securely and never commit to version control

3. **Bucket Permissions:**
   - Add `allUsers` with `Storage Object Viewer` role for public access
   - This allows direct URL access to uploaded files

#### Maven Dependencies:
The media service includes the Google Cloud Storage dependency:
```xml
<dependency>
    <groupId>com.google.cloud</groupId>
    <artifactId>google-cloud-storage</artifactId>
    <version>2.56.0</version>
</dependency>
```

#### Environment Variables for Cloud Run:
When deploying to Cloud Run, set these environment variables:
- `SPRING_PROFILES_ACTIVE=gcp` - Activates GCP profile
- `GOOGLE_APPLICATION_CREDENTIALS=/app/media-service-key.json` - Path to service account key


### Deployment Process

## Method 1: Google Cloud Console (GUI) + MongoDB Atlas (GUI)

### Step 1: Google Cloud Console Setup

#### 1.1 Enable Required APIs
1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Select project: `eca-cloud-deployment-project`
3. Navigate to **APIs & Services** → **Library**
4. Search and enable these APIs:
   - **Cloud SQL Admin API**
   - **Cloud Storage API** 
   - **Cloud Run Admin API**
   - **Cloud Build API**

#### 1.2 Create Cloud SQL Instance (MySQL)
1. Navigate to **SQL** in the left menu
2. Click **Create Instance**
3. Choose **MySQL**
4. Configure instance:
   - **Instance ID:** `course-db-instance`
   - **Password:** `<your-password>`
   - **Database version:** `MySQL 8.0`
   - **Region:** `us-central1 (Iowa)`
   - **Zone:** `us-central1-c`
5. In **Machine type and storage:**
   - **Machine type:** `db-f1-micro` (1 vCPU, 3.75 GB RAM)
   - **Storage type:** `SSD persistent disk`
   - **Storage capacity:** `10 GB`
   - **Enable storage auto-increase:**Checked**
6. Click **Create** (takes 5-10 minutes)

#### 1.3 Create Database and User
1. Once instance is ready, click on `course-db-instance`
2. Go to **Databases** tab → **Create database**
   - **Database name:** `eca_courses`
3. Go to **Users** tab → **Add user account**
   - **Username:** `<your-mysql-username>`
   - **Password:** `<your-mysql-password>`
   - **Host name:** `%` (allows from any host)

#### 1.4 Create Cloud Storage Bucket
1. Navigate to **Cloud Storage** → **Buckets**
2. Click **Create Bucket**
3. Configure bucket:
   - **Name:** `eca-cloud-deployment-media-bucket` (must be globally unique)
   - **Location type:** Regional
   - **Region:** `us-central1` (Iowa)
   - **Storage class:** Standard
   - **Access control:** Uniform
4. Click **Create**

#### 1.5 Create Service Account for Media Service
1. Go to **IAM & Admin** → **Service Accounts**
2. Click **Create Service Account**
3. Fill in details:
   - **Service account name:** `media-service-account`
   - **Description:** Service account for media service to access Cloud Storage
4. Click **Create and Continue**
5. Assign roles:
   - **Storage Object Admin** (for full bucket access)
   - **Storage Object Viewer** (for read access)
6. Click **Continue** → **Done**

#### 1.6 Download Service Account Key
1. Click on `media-service-account`
2. Go to **Keys** tab
3. Click **Add Key** → **Create new key**
4. Choose **JSON** format
5. Click **Create** - download the JSON file
6. **Important:** Keep this file secure and never commit to version control

#### 1.7 Set Bucket Permissions for Public Access
1. Go to **Cloud Storage** → **Buckets**
2. Click on `eca-cloud-deployment-media-bucket`
3. Go to **Permissions** tab
4. Click **Add Principal**
5. Add member:
   - **New members:** `allUsers`
   - **Role:** `Storage Object Viewer`
6. Click **Save**

#### 1.8 Deploy Services to Cloud Run
1. Navigate to **Cloud Run** → **Create Service**
2. For each service (course-service, student-service, media-service):
   - **Service name:** `course-service` (or respective name)
   - **Region:** `us-central1`
   - **Deploy from source:** Upload your service folder
   - **Authentication:** Allow unauthenticated invocations
   - Click **Deploy**

### Step 2: MongoDB Atlas Setup

#### 2.1 Create MongoDB Atlas Account
1. Go to [MongoDB Atlas](https://www.mongodb.com/atlas)
2. Sign up/Login with your account
3. Create a new project: `ECA Cloud Deployment`

#### 2.2 Create Free Tier Cluster
1. Click **Build a Database**
2. Choose **FREE** tier (M0 Sandbox)
3. Configure cluster:
   - **Provider:** GCP-backed
   - **Region:** `us-central1 (Iowa)`
   - **Cluster name:** `ECASTudentCluster`
4. Click **Create Cluster**

#### 2.3 Configure Database Access
1. Go to **Database Access** → **Add New Database User**
2. Create user:
   - **Username:** `<your-mongodb-username>`
   - **Password:** `<your-mongodb-password>`
   - **Database User Privileges:** `Read and write to any database`
3. Click **Add User**

#### 2.4 Configure Network Access
1. Go to **Network Access** → **Add IP Address**
2. Choose **Allow access from anywhere** (0.0.0.0/0)
3. Click **Confirm**

#### 2.5 Get Connection String
1. Go to **Database** → **Connect**
2. Choose **Connect your application**
3. Copy the connection string: `mongodb+srv://<your-mongodb-username>:<your-mongodb-password>@<your-cluster-name>.kmci8c0.mongodb.net/eca?appName=ECAStudentCluster`

## Method 2: Command Line Interface (CLI)

### Prerequisites
```bash
# Install Google Cloud SDK
curl https://sdk.cloud.google.com | bash
exec -l $SHELL

# Install MongoDB CLI (optional)
brew install mongodb/brew/mongodb-database-tools
```

### Step 1: Google Cloud Setup

#### 1.1 Authentication and Project Setup
```bash
# Login to Google Cloud
gcloud auth login

# Set default project
gcloud config set project eca-cloud-deployment-project

# Verify project
gcloud config get-value project
```

#### 1.2 Enable Required APIs
```bash
# Enable all required APIs at once
gcloud services enable sqladmin.googleapis.com storage.googleapis.com run.googleapis.com cloudbuild.googleapis.com

# Verify APIs are enabled
gcloud services list --enabled
```

#### 1.3 Create Cloud SQL Instance
```bash
# Create MySQL instance with exact specifications
gcloud sql instances create course-db-instance \
    --database-version=MYSQL_8_0 \
    --tier=db-f1-micro \
    --region=us-central1 \
    --storage-type=SSD \
    --storage-size=10GB \
    --storage-auto-increase \
    --root-password=Solution@123

# Check instance status
gcloud sql instances describe course-db-instance
```

**CLI Explanation:**
- `--database-version=MYSQL_8_0`: Specifies MySQL 8.0.41 version
- `--tier=db-f1-micro`: Sets 1 vCPU, 3.75GB RAM configuration
- `--region=us-central1`: Deploys in Iowa region for consistency
- `--storage-type=SSD`: Uses SSD for better performance
- `--storage-size=10GB`: Sets initial storage to 10GB
- `--storage-auto-increase`: Enables automatic storage scaling

#### 1.4 Create Database and User
```bash
# Create database
gcloud sql databases create eca_courses --instance=course-db-instance

# Create database user
gcloud sql users create <your-mysql-username> \
    --instance=course-db-instance \
    --password=<your-mysql-password>

# Verify database and user
gcloud sql databases list --instance=course-db-instance
gcloud sql users list --instance=course-db-instance
```

#### 1.5 Create Cloud Storage Bucket
```bash
# Create storage bucket
gsutil mb -p eca-cloud-deployment-project -c STANDARD -l us-central1 gs://eca-cloud-deployment-media-bucket

# Verify bucket creation
gsutil ls -L -b gs://eca-cloud-deployment-media-bucket
```

**CLI Explanation:**
- `gsutil mb`: Creates a new bucket
- `-p eca-cloud-deployment-project`: Specifies the project ID
- `-c STANDARD`: Sets storage class to Standard
- `-l us-central1`: Sets location to Iowa region
- `gs://eca-cloud-deployment-media-bucket`: Bucket name (must be globally unique)

#### 1.6 Create Service Account for Media Service
```bash
# Create service account
gcloud iam service-accounts create media-service-account \
    --display-name="Media Service Account" \
    --description="Service account for media service to access Cloud Storage"

# Assign Storage Object Admin role
gcloud projects add-iam-policy-binding eca-cloud-deployment-project \
    --member="serviceAccount:media-service-account@eca-cloud-deployment-project.iam.gserviceaccount.com" \
    --role="roles/storage.objectAdmin"

# Assign Storage Object Viewer role
gcloud projects add-iam-policy-binding eca-cloud-deployment-project \
    --member="serviceAccount:media-service-account@eca-cloud-deployment-project.iam.gserviceaccount.com" \
    --role="roles/storage.objectViewer"

# Create and download service account key
gcloud iam service-accounts keys create ./media-service-key.json \
    --iam-account=media-service-account@eca-cloud-deployment-project.iam.gserviceaccount.com
```

**CLI Explanation:**
- `gcloud iam service-accounts create`: Creates a new service account
- `gcloud projects add-iam-policy-binding`: Assigns IAM roles to the service account
- `roles/storage.objectAdmin`: Full access to storage objects (create, read, update, delete)
- `roles/storage.objectViewer`: Read-only access to storage objects
- `gcloud iam service-accounts keys create`: Creates and downloads the JSON key file

#### 1.7 Set Bucket Permissions for Public Access
```bash
# Make bucket publicly readable
gsutil iam ch allUsers:objectViewer gs://eca-cloud-deployment-media-bucket

# Verify bucket permissions
gsutil iam get gs://eca-cloud-deployment-media-bucket
```

**CLI Explanation:**
- `gsutil iam ch allUsers:objectViewer`: Grants public read access to all objects in the bucket
- `gsutil iam get`: Shows current IAM permissions for the bucket


### Step 2: Deploy Services to Cloud Run

#### 2.1 Deploy Course Service
```bash
# Navigate to course service
cd course-service

# Deploy to Cloud Run
gcloud run deploy course-service \
    --source . \
    --platform managed \
    --region us-central1 \
    --allow-unauthenticated \
    --set-env-vars SPRING_PROFILES_ACTIVE=gcp

# Get service URL
gcloud run services describe course-service --region=us-central1 --format="value(status.url)"
```

#### 2.2 Deploy Student Service
```bash
# Navigate to student service
cd ../student-service

# Deploy to Cloud Run
gcloud run deploy student-service \
    --source . \
    --platform managed \
    --region us-central1 \
    --allow-unauthenticated \
    --set-env-vars SPRING_PROFILES_ACTIVE=gcp

# Get service URL
gcloud run services describe student-service --region=us-central1 --format="value(status.url)"
```

#### 2.3 Deploy Media Service
```bash
# Navigate to media service
cd ../media-service

# Deploy to Cloud Run with service account key
gcloud run deploy media-service \
    --source . \
    --platform managed \
    --region us-central1 \
    --allow-unauthenticated \
    --set-env-vars SPRING_PROFILES_ACTIVE=gcp \
    --set-env-vars GOOGLE_APPLICATION_CREDENTIALS=/app/media-service-key.json

# Upload service account key as a secret
gcloud run services update media-service \
    --region=us-central1 \
    --update-secrets=GOOGLE_APPLICATION_CREDENTIALS=media-service-key:latest

# Get service URL
gcloud run services describe media-service --region=us-central1 --format="value(status.url)"
```

**CLI Explanation:**
- `--set-env-vars SPRING_PROFILES_ACTIVE=gcp`: Activates GCP profile for Cloud Storage integration
- `--set-env-vars GOOGLE_APPLICATION_CREDENTIALS`: Points to the service account key file
- `--update-secrets`: Securely mounts the service account key as a secret in the container


**CLI Explanation:**
- `--source .`: Builds and deploys from current directory
- `--platform managed`: Uses fully managed Cloud Run
- `--region us-central1`: Deploys in Iowa region
- `--allow-unauthenticated`: Makes service publicly accessible
- `--set-env-vars SPRING_PROFILES_ACTIVE=gcp`: Activates GCP profile


### Why This Way?

1. **Microservices on Cloud Run:** Serverless, auto-scaling, pay-per-use model
2. **Managed Databases:** Cloud SQL and MongoDB Atlas handle backups, updates, and maintenance
3. **Cloud Storage Integration:** Google Cloud Storage provides scalable, durable file storage with global CDN
4. **Regional Consistency:** All services in us-central1 for low latency and data locality
5. **Security:** Cloud SQL proxy, MongoDB SRV connections, and service account authentication ensure secure data transmission
6. **Cost Optimization:** Free tier MongoDB, minimal Cloud SQL instance, serverless compute, and pay-per-use storage
7. **Public Access:** Direct URL access to media files without authentication overhead for better performance

