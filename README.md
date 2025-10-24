# Google Skills Tutorials Collection 🎓

A comprehensive collection of hands-on tutorials and labs for Google Cloud Platform (GCP) services and technologies. This repository contains practical examples, code samples, and step-by-step guides to help you master various Google Cloud skills.

[![Python Version](https://img.shields.io/badge/python-3.13%2B-blue.svg)](https://www.python.org/downloads/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Google Cloud](https://img.shields.io/badge/Google%20Cloud-Platform-4285F4?logo=google-cloud)](https://cloud.google.com/)

---

## 📚 Table of Contents

- [Overview](#overview)
- [Available Tutorials](#available-tutorials)
- [Getting Started](#getting-started)
- [Repository Structure](#repository-structure)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
- [Contributing](#contributing)
- [Resources](#resources)
- [License](#license)

---

## 🌟 Overview

This repository serves as a centralized hub for Google Cloud Platform tutorials, covering a wide range of services including:

- **Machine Learning & AI** - Vertex AI, TensorFlow, AutoML
- **Data Analytics** - BigQuery, Dataflow, Dataproc
- **Cloud Infrastructure** - Compute Engine, Cloud Storage, Networking
- **Serverless & Containers** - Cloud Functions, Cloud Run, GKE
- **DevOps & CI/CD** - Cloud Build, Artifact Registry, Cloud Deploy
- **Databases** - Cloud SQL, Firestore, Bigtable
- **Security & Identity** - IAM, Secret Manager, Cloud Armor

Each tutorial is self-contained with detailed documentation, code samples, and practical exercises.

---

## 📖 Available Tutorials

### Machine Learning & AI

| Tutorial                                                                                                            | Description                                                                          | Lab ID | Difficulty   |
| ------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ | ------ | ------------ |
| [**Machine Learning with TensorFlow and Vertex AI**](./Machine%20Learning%20with%20TensorFlow%20and%20Vertex%20AI/) | Build, train, and deploy TensorFlow models on Vertex AI with explainability features | GSP273 | Intermediate |

### Data Analytics

_Coming soon..._

### Cloud Infrastructure

_Coming soon..._

### Serverless & Containers

_Coming soon..._

---

## 🚀 Getting Started

### Quick Start

1. **Clone the repository**

   ```bash
   git clone https://github.com/bigalex95/google-skills.git
   cd google-skills
   ```

2. **Set up Python environment**

   ```bash
   # Using uv (recommended)
   uv venv
   source .venv/bin/activate  # On Windows: .venv\Scripts\activate

   # Or using traditional venv
   python -m venv .venv
   source .venv/bin/activate
   ```

3. **Install dependencies**

   ```bash
   uv pip install -r requirements.txt
   ```

4. **Choose a tutorial and follow its README**
   ```bash
   cd "Machine Learning with TensorFlow and Vertex AI"
   ```

---

## 📁 Repository Structure

```
google-skills/
├── README.md                                      # This file
├── pyproject.toml                                 # Project configuration
├── requirements.txt                               # Python dependencies
├── .gitignore                                     # Git ignore rules
├── .python-version                                # Python version specification
│
├── Machine Learning with TensorFlow and Vertex AI/
│   ├── README.md                                  # Tutorial documentation
│   ├── main.ipynb                                 # Jupyter notebook
│   └── ...                                        # Additional resources
│
├── [Future Tutorial 1]/
│   ├── README.md
│   └── ...
│
└── [Future Tutorial 2]/
    ├── README.md
    └── ...
```

Each tutorial directory contains:

- **README.md** - Detailed tutorial instructions
- **Code samples** - Python scripts, notebooks, or configuration files
- **Resources** - Additional documentation, datasets, or assets

---

## ✅ Prerequisites

Before starting these tutorials, ensure you have:

### Required

- **Google Cloud Account** - [Sign up here](https://cloud.google.com/free)
- **Python 3.13+** - [Download here](https://www.python.org/downloads/)
- **Git** - [Install here](https://git-scm.com/downloads)

### Recommended

- **Google Cloud SDK (gcloud CLI)** - [Install guide](https://cloud.google.com/sdk/docs/install)
- **VS Code** or **JupyterLab** - For working with notebooks
- **Basic knowledge of:**
  - Python programming
  - Cloud computing concepts
  - Command line interface (CLI)

### Google Cloud Setup

1. Create a Google Cloud account (free tier available)
2. Create a new project or use an existing one
3. Enable billing (required for most services)
4. Install and initialize gcloud CLI:
   ```bash
   gcloud init
   gcloud auth login
   ```

---

## 💻 Installation

### Option 1: Using uv (Recommended)

```bash
# Install uv if you haven't already
curl -LsSf https://astral.sh/uv/install.sh | sh

# Clone the repository
git clone https://github.com/bigalex95/google-skills.git
cd google-skills

# Create virtual environment and install dependencies
uv venv
source .venv/bin/activate
uv pip install -r requirements.txt
```

### Option 2: Using pip

```bash
# Clone the repository
git clone https://github.com/bigalex95/google-skills.git
cd google-skills

# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### Option 3: Using conda

```bash
# Clone the repository
git clone https://github.com/bigalex95/google-skills.git
cd google-skills

# Create conda environment
conda create -n google-skills python=3.13
conda activate google-skills

# Install dependencies
pip install -r requirements.txt
```

---

## 🎯 Usage

### Working with Jupyter Notebooks

Many tutorials include Jupyter notebooks for interactive learning:

```bash
# Start Jupyter Lab
jupyter lab

# Or Jupyter Notebook
jupyter notebook
```

Then navigate to the tutorial directory and open the `.ipynb` file.

### Running Python Scripts

For standalone Python scripts:

```bash
# Navigate to tutorial directory
cd "Machine Learning with TensorFlow and Vertex AI"

# Run the script
python main.py
```

### Using Google Cloud Shell

You can also run these tutorials directly in Google Cloud Shell:

1. Open [Google Cloud Console](https://console.cloud.google.com/)
2. Click the Cloud Shell icon (top right)
3. Clone the repository and follow the tutorial instructions

---

## 🤝 Contributing

Contributions are welcome! If you'd like to add a new tutorial or improve existing ones:

1. **Fork the repository**

2. **Create a feature branch**

   ```bash
   git checkout -b feature/new-tutorial
   ```

3. **Add your tutorial** following this structure:

   ```
   Your Tutorial Name/
   ├── README.md          # Detailed tutorial guide
   ├── main.ipynb         # Jupyter notebook (if applicable)
   ├── requirements.txt   # Tutorial-specific dependencies
   └── assets/            # Images, datasets, etc.
   ```

4. **Commit your changes**

   ```bash
   git add .
   git commit -m "Add: Tutorial on [topic]"
   ```

5. **Push to your fork**

   ```bash
   git push origin feature/new-tutorial
   ```

6. **Merge to master (for maintainers)**

   ```bash
   # Switch to master branch
   git checkout master

   # Pull latest changes
   git pull origin master

   # Merge your feature branch
   git merge feature/new-tutorial

   # Resolve any conflicts if they occur
   # Then commit the merge

   # Push to remote master
   git push origin master

   # Delete the feature branch (optional)
   git branch -d feature/new-tutorial
   ```

7. **Create a Pull Request (for contributors)**
   - Go to your forked repository on GitHub
   - Click "Compare & pull request"
   - Fill in the PR description with:
     - What tutorial you added/changed
     - Testing you performed
     - Any special notes
   - Submit the pull request for review

### Tutorial Guidelines

When creating a new tutorial, please ensure:

- Clear, step-by-step instructions
- Working code samples tested on GCP
- Proper error handling and troubleshooting sections
- Estimated completion time
- Cost estimates (if applicable)
- Clean up instructions to avoid unnecessary charges

---

## 📚 Resources

### Official Documentation

- [Google Cloud Documentation](https://cloud.google.com/docs)
- [Google Cloud Skills Boost](https://www.cloudskillsboost.google/)
- [Google Cloud Blog](https://cloud.google.com/blog)
- [Google Cloud YouTube Channel](https://www.youtube.com/googlecloud)

### Learning Paths

- [Google Cloud Training](https://cloud.google.com/training)
- [Google Cloud Certifications](https://cloud.google.com/certification)
- [Coursera - Google Cloud Courses](https://www.coursera.org/googlecloud)

### Community

- [Google Cloud Community](https://www.googlecloudcommunity.com/)
- [Stack Overflow - google-cloud-platform](https://stackoverflow.com/questions/tagged/google-cloud-platform)
- [Reddit - r/googlecloud](https://www.reddit.com/r/googlecloud/)

### Books

- _Data Science on the Google Cloud Platform, 2nd Edition_ by Valliappa Lakshmanan
- _Google Cloud Platform for Architects_ by Vitthal Srinivasan
- _Google Cloud Platform Cookbook_ by Legorie Rajan PS

---

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## ⭐ Acknowledgments

- Google Cloud Platform for providing excellent documentation and labs
- The open-source community for tools and libraries
- Contributors who help improve these tutorials

---

## 📧 Contact & Support

- **Issues**: [GitHub Issues](https://github.com/bigalex95/google-skills/issues)
- **Discussions**: [GitHub Discussions](https://github.com/bigalex95/google-skills/discussions)
- **Email**: ear.cv.dev@gmail.com

---

## 🗺️ Roadmap

### Upcoming Tutorials

- [ ] BigQuery for Data Analysis
- [ ] Cloud Functions Serverless Applications
- [ ] Kubernetes Engine (GKE) Deployment
- [ ] Cloud SQL Database Management
- [ ] Cloud Storage Best Practices
- [ ] Cloud Run Containerized Applications
- [ ] Pub/Sub Messaging Patterns
- [ ] Cloud Dataflow Stream Processing
- [ ] Cloud Composer Workflow Orchestration
- [ ] Cloud Build CI/CD Pipelines

---

<div align="center">

**Happy Learning!** 🚀

If you find this repository helpful, please consider giving it a ⭐

[Report Bug](https://github.com/bigalex95/google-skills/issues) · [Request Feature](https://github.com/bigalex95/google-skills/issues) · [Contribute](https://github.com/bigalex95/google-skills/pulls)

</div>
