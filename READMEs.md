# 📦 S3-Compatible Storage Setup Guide (Cloudflare R2 or AWS S3)

This guide explains how to configure and use an S3-compatible storage system like **Cloudflare R2** or **AWS S3** in your application, including bucket creation, credentials setup, CORS configuration, lifecycle rules, and example environment variables.

---

## ✅ Step 1: Create a Bucket

* Go to your storage provider's dashboard (e.g., Cloudflare R2 or AWS S3).
* Create a **new bucket**.
* Choose a unique **bucket name**, e.g., `my-app-assets`.
* Set the **region** (for R2, use `auto`).

---

## ✅ Step 2: Generate Access Credentials

Create or use existing access credentials:

| Field      | Description                                                    |
| ---------- | -------------------------------------------------------------- |
| Access Key | Public identifier for API access                               |
| Secret Key | Secret used to sign requests                                   |
| Endpoint   | API URL (e.g. `https://<account-id>.r2.cloudflarestorage.com`) |

Store securely using environment variables:

```env
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key
AWS_REGION=auto
AWS_S3_ENDPOINT=https://<account-id>.r2.cloudflarestorage.com
AWS_BUCKET_NAME=my-app-assets
```

---

## ✅ Step 3: Configure CORS Policy

Enable CORS so your frontend can communicate with the S3 bucket.

### CORS JSON Configuration

```json
[
  {
    "AllowedOrigins": ["*"],
    "AllowedMethods": ["GET", "PUT", "POST"],
    "AllowedHeaders": ["*"],
    "ExposeHeaders": ["ETag"]
  }
]
```

### Apply with AWS CLI (for Cloudflare R2)

Save the above JSON as `cors.json` and run:

```bash
aws s3api put-bucket-cors \
  --bucket my-app-assets \
  --region auto \
  --endpoint-url https://<account-id>.r2.cloudflarestorage.com \
  --cors-configuration file://cors.json
```

---

## ✅ Step 4: Optional - Object Lifecycle Rules

You can define rules for managing stored objects automatically.

### Example Rule

| Rule Name               | Prefix | Action                     | Status  |
| ----------------------- | ------ | -------------------------- | ------- |
| Default Multipart Abort | --     | Abort uploads after 7 days | Enabled |

---

## ✅ Step 5: Optional - Bucket Lock Rules

You can lock objects to prevent deletion for a retention period (use carefully).

* Only apply if regulatory compliance requires this.
* Prevents overwriting and deletions until the lock expires.

---

## ✅ Summary: Required Configuration

| Item            | Example Value                                   |
| --------------- | ----------------------------------------------- |
| Bucket Name     | `my-app-assets`                                 |
| Access Key      | `AKIA...`                                       |
| Secret Key      | `abcd1234...`                                   |
| Endpoint        | `https://<account-id>.r2.cloudflarestorage.com` |
| Region          | `auto`                                          |
| Allowed Methods | `GET`, `PUT`, `POST`                            |
| Allowed Headers | `*`                                             |
| Allowed Origins | `*`                                             |
| Expose Headers  | `ETag`                                          |

---

## ✅ Bun Integration (Example Code)

```ts
import { s3, write } from "bun";

const file = s3.file("uploads/example.json");

// Upload JSON data
await write(file, JSON.stringify({ message: "Hello from Bun!" }));

// Read JSON data back
const data = await file.json();
console.log(data);

// Generate pre-signed URL
const url = file.presign({ expiresIn: 60 * 60 * 24 }); // 1 day
console.log(url);

// Delete file
await file.delete();
```

---

## 📄 Done!

You can now upload, read, presign, and delete files using your configured S3-compatible bucket with full CORS and lifecycle support.
