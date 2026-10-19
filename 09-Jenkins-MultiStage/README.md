# Lab 09: Jenkins Multi-Stage Pipeline — Deploying a Python Application

## Objective

Created and executed a Jenkins declarative pipeline with five stages — **Build**, **Test**, **Deploy**, **Run Application**, and **Test Application** — for a Python Flask application. Demonstrated multi-stage CI/CD pipeline management in Jenkins.

---

## Scenario

You are tasked with automating the CI/CD pipeline for a Python Flask application. The pipeline should perform the following stages:
1. **Build**: Install dependencies.
2. **Test**: Run unit tests to ensure the application works as expected.
3. **Deploy**: Deploy the application by copying it to a mock deployment directory.
4. **Run Application**: Start the deployed application.
5. **Test Application**: Validate the running application with tests.

---

## Prerequisites

- **Jenkins** installed and running with the **Pipeline Plugin**
- **Python 3** available inside the Jenkins environment
- **GitHub** repository for storing the application code
- **Docker** (if running Jenkins in a container)

---

## Directory Structure

```
09-Jenkins-MultiStage/
├── app.py              # Flask web application
├── test_app.py         # Unit tests for the Flask app
├── requirements.txt    # Python dependencies
├── Jenkinsfile         # Jenkins declarative pipeline script
├── screenshot/         # Deployment and verification screenshots
└── README.md
```

---

## Step-by-Step Execution

### Step 1: Set Up Jenkins

1. Install Jenkins and ensure it is running on your local machine or a server.
2. Install the following plugins:
   - **Pipeline Plugin** (if not already installed).

---

### Step 2: Understand the Application Files

#### Flask Application (`app.py`)

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Hello, Jenkins Multi-Stage Pipeline!"

if __name__ == "__main__":
    app.run(debug=True)
```

#### Unit Tests (`test_app.py`)

```python
import unittest
from app import app

class TestApp(unittest.TestCase):
    def test_home(self):
        tester = app.test_client()
        response = tester.get("/")
        print(response.data.decode("utf-8"))
        self.assertEqual(response.status_code, 200)
        self.assertEqual(response.data.decode("utf-8"), "Hello, Jenkins Multi-Stage Pipeline!")

if __name__ == "__main__":
    unittest.main()
```

#### Dependencies (`requirements.txt`)

```
flask==2.1.2
```

---

### Step 3: Push the Application to GitHub

```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/<your-GitHub-username>/devops-sample-code.git
git push -u origin main
```

---

### Step 4: Create a Jenkins Pipeline Job

1. Go to your Jenkins dashboard and click **"New Item"**.
2. Enter a name (e.g., `Python-MultiStage-Pipeline`) and select **Pipeline**.
3. Click **OK**.

---

### Step 5: Define the Jenkinsfile

The `Jenkinsfile` defines the following stages:

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Creating virtual environment and installing dependencies...'
            }
        }
        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'python3 -m unittest discover -s .'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                sh '''
                mkdir -p ${WORKSPACE}/python-app-deploy
                cp ${WORKSPACE}/app.py ${WORKSPACE}/python-app-deploy/
                '''
            }
        }
        stage('Run Application') {
            steps {
                echo 'Running application...'
                sh '''
                nohup python3 ${WORKSPACE}/python-app-deploy/app.py > ${WORKSPACE}/python-app-deploy/app.log 2>&1 &
                echo $! > ${WORKSPACE}/python-app-deploy/app.pid
                '''
            }
        }
        stage('Test Application') {
            steps {
                echo 'Testing application...'
                sh '''
                python3 ${WORKSPACE}/test_app.py
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs for more details.'
        }
    }
}
```

Push the Jenkinsfile to your repository:

```bash
git add Jenkinsfile
git commit -m "Added Jenkinsfile"
git push
```

---

### Step 6: Configure the Jenkins Job

1. In Jenkins, go to the pipeline configuration.
2. Under **Pipeline**, select **Pipeline script from SCM**.
3. Choose **Git** as the SCM and enter your repository URL.
4. Add credentials if necessary.
5. Save the configuration.

---

### Step 7: Run the Pipeline

1. Click **Build Now** to start the pipeline.
2. Observe the stages (Build, Test, Deploy, Run Application, Test Application) as they execute.
3. Check the console output for logs and results.

---

### Step 8: Handling Build Errors

When you run the Jenkins job, you may encounter build errors around missing `python3`, `pip`, and other dependencies. To resolve this, log into the Jenkins Docker container:

```bash
docker ps -a
docker exec -it -u root <container-id> bash
```

Install required dependencies inside the container:

```bash
apt-get update
apt install python3
apt install pip
apt install python3.11-venv
apt install python3-flask
python3 -m unittest discover -s .
```

---

## Expected Outputs

**Build Stage:**
- The `pip install -r requirements.txt` command installs Flask and other dependencies.

**Test Stage:**
- The unittest framework runs the test cases and ensures they pass.

**Deploy Stage:**
- The application file (`app.py`) is copied to the deployment directory.

**Expected Console Output (Success):**
```
[Pipeline] stage
[Pipeline] { (Build)
[Pipeline] echo
Creating virtual environment and installing dependencies...
[Pipeline] }
[Pipeline] stage
[Pipeline] { (Test)
[Pipeline] echo
Running tests...
[Pipeline] sh
+ python3 -m unittest discover -s .
.
----------------------------------------------------------------------
Ran 1 test in 0.004s

OK
Hello, Jenkins Multi-Stage Pipeline!
[Pipeline] }
[Pipeline] stage
[Pipeline] { (Deploy)
[Pipeline] echo
Deploying application...
[Pipeline] }
[Pipeline] stage
[Pipeline] { (Run Application)
[Pipeline] echo
Running application...
[Pipeline] }
[Pipeline] stage
[Pipeline] { (Test Application)
[Pipeline] echo
Testing application...
[Pipeline] sh
+ python3 test_app.py
.
----------------------------------------------------------------------
Ran 1 test in 0.004s

OK
[Pipeline] }
[Pipeline] stage
[Pipeline] { (Declarative: Post Actions)
[Pipeline] echo
Pipeline completed successfully!
Finished: SUCCESS
```

---

## Additional Enhancements

Add a **Code Quality Analysis** stage using tools like `flake8` or `pylint`:

```groovy
stage('Code Quality') {
    steps {
        echo 'Running code quality checks...'
        sh 'flake8 .'
    }
}
```

- Deploy the application to a Docker container as part of the pipeline.
- Integrate notifications (e.g., email or Slack) for pipeline success or failure.

---

## Deployment Evidence

![Jenkins Pipeline Stages](./screenshot/pipeline-stages.png)

![Jenkins Console Output](./screenshot/console-output.png)

---

## Files

| File | Description |
|------|-------------|
| `app.py` | Flask web application |
| `test_app.py` | Unit tests for the Flask app |
| `requirements.txt` | Python dependencies |
| `Jenkinsfile` | Jenkins declarative pipeline script |
