# Accessing UK Biobank RAP via CLI on HPC

This guide helps you configure and use the **DNAnexus CLI** on a High Performance Computing (HPC) environment to interact with the **UK Biobank Research Analysis Platform (RAP)**.

---

## 🧰 1. Load Required Modules

```bash
module load GCCcore/14.2.0
module load Python/3.13.1
```

pip install --upgrade pip
pip install --upgrade dxpy
pip install readline  # Optional but recommended for CLI usability
