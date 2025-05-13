Here’s a comprehensive `README.md` file based on the lab **"Cloud IAM: Qwik Start"**, with detailed explanations added for each step.

---

````markdown
# 🔐 Cloud IAM: Qwik Start

This guide walks you through the basics of **Google Cloud IAM (Identity and Access Management)** using the Google Cloud Console. You'll explore how to grant, restrict, and tailor access to specific services such as Cloud Storage using predefined IAM roles.

---

## 📘 Overview

**Cloud IAM** allows you to manage access control by defining who (identity) has what access (roles) to which resources. This lab will help you understand:

- How to view and manage IAM roles
- The difference between primitive and predefined roles
- How IAM changes affect user permissions
- How to assign granular access to services like Cloud Storage

---

## 🧰 Prerequisites

- A Google Cloud Platform (GCP) project with two user accounts: **Username 1** (Owner) and **Username 2** (Viewer)
- Access to Google Cloud Console
- Basic understanding of cloud concepts

---

## 🎯 Objectives

1. Explore project-level IAM roles
2. Create a Cloud Storage bucket and upload a file
3. Test and revoke user access
4. Assign resource-specific permissions

---

## 🛠 Task 1: Explore IAM Console & Project Roles

### 🔎 View IAM Roles

1. Sign in as **Username 1**.
2. Navigate to:  
   `Navigation menu > IAM & Admin > IAM`
3. Click **+GRANT ACCESS** to open the role assignment panel.
4. Scroll to **Basic** roles:
   - **Viewer** – Read-only access (cannot make changes)
   - **Editor** – Can view and modify resources
   - **Owner** – All editor permissions + manage roles, billing

   > These are known as **primitive roles**. They apply at the project level and affect all services unless restricted further.

5. Click **CANCEL** to exit.

### 👀 Compare with Username 2

1. Switch to the **Username 2** console.
2. Go to `IAM & Admin > IAM`.
3. Look for both users:
   - Username 1 should have the **Owner** role.
   - Username 2 should have the **Viewer** role.
4. Notice:
   - The **+GRANT ACCESS** button is disabled.
   - Hovering shows:  
     *“You need permissions for this action…”*  
     This illustrates how roles directly affect user capabilities.

---

## ☁️ Task 2: Prepare Cloud Storage Bucket for Access Testing

### 🪣 Create a Storage Bucket

1. Switch to **Username 1**.
2. Navigate to:  
   `Navigation menu > Cloud Storage > Buckets`
3. Click **+CREATE**.
4. Configure:
   - **Name**: Choose a globally unique name (e.g., `my-iam-lab-bucket`)
   - **Location type**: `Multi-region`
   - Leave other settings as default.
5. Click **CREATE**.
6. If prompted to block public access, click **Confirm**.

> ⚠️ If you see a permission error, recheck that you're signed in as Username 1.

### 📤 Upload a Sample File

1. On the bucket page, click **UPLOAD FILES**.
2. Select any file (e.g., `.txt` or `.html`) from your system.
3. Click the three dots beside the uploaded file, select **Rename**, and rename it to:  
   `sample.txt`

---

## 👁 Task 3: Verify Access as Viewer

1. Switch to **Username 2**.
2. Navigate to:  
   `Navigation menu > Cloud Storage > Buckets`
3. You should be able to **see the bucket and `sample.txt` file**, but not perform actions like editing or deleting.

> This demonstrates how the **Viewer role** allows read-only access at the project level.

---

## 🚫 Task 4: Remove Project Access

### 🔐 Revoke Access for Username 2

1. Switch to **Username 1**.
2. Go to `IAM & Admin > IAM`.
3. Find **Username 2** and click the ✏️ pencil icon.
4. Remove the **Viewer** role using the 🗑 trashcan icon.
5. Click **SAVE**.

> Note: It may take **up to 80 seconds** for changes to propagate across GCP systems.

### ❌ Verify Access Removal

1. Switch back to **Username 2**.
2. Go to `Navigation menu > Cloud Storage > Buckets`.
3. You should see a **permission error**, indicating access has been revoked.

> 🔁 Refresh the page after a minute if the error hasn't appeared yet.

---

## ✅ Task 5: Add Storage-Specific Access

### 🎯 Grant Storage Object Viewer Role

1. Switch back to **Username 1**.
2. Navigate to:  
   `IAM & Admin > IAM`
3. Click **+GRANT ACCESS**.
4. Enter **Username 2** under **New principals**.
5. Under **Select a role**, choose:  
   `Cloud Storage > Storage Object Viewer`
6. Click **SAVE**.

> This assigns **fine-grained access** to a specific resource type (Cloud Storage), rather than entire project access.

---

## 🔍 Verify Access via Cloud Shell

1. Switch to **Username 2**.
2. Go to Cloud Console > **Activate Cloud Shell**.
3. In the shell, run:

```bash
gsutil ls gs://[YOUR_BUCKET_NAME]
````

> Replace `[YOUR_BUCKET_NAME]` with the bucket you created earlier.

You should see:

```bash
gs://[YOUR_BUCKET_NAME]/sample.txt
```

> 🕒 If you get an `AccessDeniedException`, wait 30–60 seconds and try again.

This confirms that even without **project-level access**, Username 2 can now view **Cloud Storage contents** thanks to the **Storage Object Viewer** role.

---

## 🏁 Summary

In this lab, you:

* Explored basic IAM roles: Owner, Editor, Viewer
* Observed how role assignments affect user capabilities
* Learned the difference between **project-level** and **resource-level** access
* Managed Cloud Storage permissions using fine-grained IAM roles


## 🔗 Useful Links

* [Cloud IAM Overview](https://cloud.google.com/iam/docs/overview)
* [Basic IAM Roles](https://cloud.google.com/iam/docs/understanding-roles#primitive_roles)
* [Cloud Storage Access Control](https://cloud.google.com/storage/docs/access-control/iam)
* [gsutil CLI Tool](https://cloud.google.com/storage/docs/gsutil)
