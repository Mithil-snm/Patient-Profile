# Patient Registration & File Upload — Wellytics Platform

A Jupyter Notebook to bulk register patient profiles on the Wellytics platform and automatically upload clinical files (documents and audio) to the correct patient folders.

---

## Features

- Bulk patient registration via CSV
- Duplicate detection — if a patient already exists, files are still uploaded
- Upload-only mode — enter just the `Patient Id` in the CSV to skip registration and upload files directly
- Auto-fetches patient name, doctor ID, and org ID from the platform for upload-only rows
- Smart file matching — matches files to patients by name (e.g. `aaravsharma1.pdf` → Aarav Sharma)
- Audio files (`.mp3`, `.wav`, `.m4a`, etc.) → uploaded to **TRANSCRIPTION** folder with drug detection ON
- All other files (`.pdf`, `.doc`, etc.) → uploaded to **CLINICAL_NOTES** folder
- Supports file upload from **local folder**, **Google Drive**, or **both**
- Works across DEV / STAGE / DEMO / PROD environments

---

## Requirements

### Python packages
```bash
pip install pandas requests google-api-python-client google-auth
```

### Jupyter
```bash
pip install notebook
```

---

## CSV Format

The notebook reads from a CSV file. Each row is either a **new patient** or an **upload-only** row.

### New Patient Row
All mandatory fields must be filled. `Patient Id` must be left blank.

| Column | Required | Notes |
|---|---|---|
| First Name | ✅ | |
| Last Name | ✅ | |
| Age | ✅ | |
| Gender | ✅ | MALE / FEMALE |
| Mobile Number | ✅ | |
| Doctor Id | ✅ | UUID from platform |
| Organization Id | ✅ | UUID from platform |
| Email | ❌ | Optional |
| Country Code | ❌ | Defaults to `+91` |
| Type | ❌ | Defaults to `OP` |
| DOB | ❌ | |
| Blood Group | ❌ | |
| Marital Status | ❌ | |
| External Id | ❌ | |
| ABHA Id | ❌ | |
| Rh Type | ❌ | |
| Patient Id | ❌ | Leave blank for new registration |

### Upload-Only Row
Only `Patient Id` is required. All other fields can be left blank. The notebook will automatically fetch the patient's name, doctor, and organisation from the platform.

| Column | Required |
|---|---|
| Patient Id | ✅ |
| Everything else | ❌ Leave blank |

### Example CSV

```
First Name,Last Name,Age,Gender,Mobile Number,Email,Doctor Id,Organization Id,Patient Id
Arjun,Verma,34,Male,9876543210,arjun@example.com,doctor-uuid,org-uuid,
Sneha,Reddy,28,Female,9876543211,sneha@example.com,doctor-uuid,org-uuid,
,,,,,,,,existing-patient-uuid-here
,,,,,,,,another-patient-uuid-here
```

---

## File Naming Convention

Files must be named after the patient's full name (lowercase, no spaces):

| Patient Name | Valid File Names |
|---|---|
| Aarav Sharma | `aaravsharma.pdf`, `aaravsharma1.mp3`, `aaravsharma2.doc` |
| Priya Reddy | `priyareddy.pdf`, `priyareddy1.wav` |

Trailing numbers are ignored during matching — `aaravsharma3.mp3` still maps to Aarav Sharma.

---

## File Upload Routing

| File Type | Endpoint | Drug Detection |
|---|---|---|
| `.mp3`, `.wav`, `.m4a`, `.ogg`, `.flac`, `.aac`, `.wma`, `.aiff` | TRANSCRIPTION | ✅ Always ON |
| All other files (`.pdf`, `.doc`, `.jpg`, etc.) | CLINICAL_NOTES | — |

---

## Configuration

Open the notebook and update **Cell 1** before each run:

```python
ENV                = "DEV"       # DEV / STAGE / DEMO / PROD
filepath           = "/path/to/PatientProfiles.csv"
FILE_SOURCE        = "none"      # "local" / "drive" / "both" / "none"
LOCAL_FILES_FOLDER = ""          # path to local files folder (if FILE_SOURCE is local/both)
```

### FILE_SOURCE options

| Value | Behaviour |
|---|---|
| `"none"` | Register patients only, no file upload |
| `"local"` | Upload files from local folder only |
| `"drive"` | Upload files from Google Drive only |
| `"both"` | Upload files from both local and Google Drive |

---

## Google Drive Setup

To upload files from Google Drive, a Google Service Account is required. The credentials are already embedded in the notebook (Cell 2). The Drive folder ID is also pre-configured.

To use a different Drive folder:
1. Share the folder with the service account email:
   `agent-236@exalted-analogy-477207-g8.iam.gserviceaccount.com`
2. Update `DRIVE_FOLDER_ID` in Cell 2

---

## How to Run

1. Install dependencies:
```bash
pip install pandas requests google-api-python-client google-auth
```

2. Fill in your CSV file

3. Open the notebook:
```bash
jupyter notebook Patient_Registration.ipynb
```

4. Update **Cell 1** with your ENV, CSV path, and FILE_SOURCE

5. Run all cells top to bottom (**Kernel → Restart & Run All**)

---

## Output Summary

After running, the notebook prints a summary:

```
=============================================
              FINAL SUMMARY
=============================================

  PATIENTS
  ✅ Newly Registered : 3
  ⚠️  Already Existed  : 2
  ❌ Failed           : 0

  FILES (6 found)
  ✅ Uploaded         : 6
  ⚠️  No Match         : 0
  ❌ Failed           : 0

  📂 Patients with files: 4
=============================================
```

---

## Environments

| ENV | Base URL |
|---|---|
| DEV | `https://api-dev.platform.wellytics.health/auth/api` |
| STAGE / DEMO | `https://api-stage.platform.wellytics.health/auth/api` |
| PROD | `https://api.platform.wellytics.health/auth/api` |

---

## Notes

- Patient IDs are environment-specific — a patient registered in DEV will not exist in STAGE
- If a mobile number is already registered, the existing patient ID is used and files are still uploaded
- Audio files are always uploaded with `transcriptionWithDrugDetection=true`
- Each file upload generates a unique filename using UUID to avoid overwrites
