# google_skills

My notebook and notes for one Google Cloud Skills Boost lab: **Machine Learning with TensorFlow and Vertex AI (GSP273)**. Learning repo, not maintained.

## What's inside

```
Machine Learning with TensorFlow and Vertex AI/
  README.md          Lab instructions (copied from the lab page)
  main.ipynb         Notebook worked through during the lab
  requirements.txt   Python dependencies for running the notebook locally
main.py              Placeholder script from `uv init` (prints a greeting)
pyproject.toml       uv project metadata (Python >= 3.13, no dependencies listed)
```

The lab covers building a TensorFlow model in a Jupyter notebook on Vertex AI Workbench, deploying it to a Vertex AI endpoint, and requesting predictions. No other labs or services are included.

## Run

The notebook was written for the lab's Vertex AI Workbench environment and needs a Google Cloud project to run end to end. To open it locally:

```bash
cd "Machine Learning with TensorFlow and Vertex AI"
pip install -r requirements.txt
jupyter lab main.ipynb
```

Cells that call Google Cloud APIs (BigQuery, Vertex AI) will fail without credentials and a project. I did not re-run the notebook locally.

## Status

A single lab, kept for reference.

## Credits

The lab and its code samples are from Google Cloud Skills Boost, which uses material from *Data Science on the Google Cloud Platform, 2nd Edition* (O'Reilly Media). They belong to their owners; this repo holds my working copy only.
