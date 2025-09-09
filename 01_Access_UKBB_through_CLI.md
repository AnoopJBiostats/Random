# Accessing UK Biobank RAP via CLI on HPC

This guide helps you configure and use the **DNAnexus CLI** on a High Performance Computing (HPC) environment to interact with the **UK Biobank Research Analysis Platform (RAP)**.

---

## 1. Load Required Modules

```bash
module load GCCcore/14.2.0
module load Python/3.13.1
```

##  2. Install DNAnexus SDK (dxpy) - dna nexus toolkit

```bash
pip install dxpy
pip install readline  # Optional but recommended for CLI usability
```

## 3. Authenticate with DNAnexus
```bash
# check this
dx login  # Follow prompts to log in
dx whoami  # Check if logged in correctly
```

# 4. Explore Project Structure
```bash
dx ls
dx pwd
dx cd Project_Clinical_info/
dx ls -la
```

## 5. Upload/Download/Test Files
```bash
echo "Hello_PGx!" > testing1.txt
dx upload testing1.txt
dx ls
dx download testing1.txt -o ~/Testing_CLI/testing1.txt
dx cat testing1.txt
dx describe testing1.txt
```

## 6. Select and Manage Projects
```bash
dx select  # Choose a project
dx select --level VIEW
dx select --level ADMINISTRATOR
```

## 7. Search for Data
```bash
dx find data --name "file_name.txt"
dx find data --property eid=1000023
dx find data --property field_id=22022
```

## 8. Help & Documentation
```bash
dx --help
dx help all
dx help exec
dx help run
```




