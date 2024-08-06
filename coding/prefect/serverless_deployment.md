# 🚀 Deploying Python Functions on Cloud Run with Prefect

This guide will help you deploy Python functions on Google Cloud Run using Prefect, making your workflows scalable and easy to manage.

## 📋 Prerequisites

- Prefect Cloud account (free tier is fine)
- Google Cloud Platform account (might incur in charges)
- Python environment (preferably a conda environment)
- gcloud CLI

## 🛠️ Setup

1. **Install Prefect**
   ```bash
   conda activate yourvenv
   conda install prefect
   ```

2. **Install gcloud CLI**
   Follow the [official docs](https://cloud.google.com/sdk/docs/install-sdk) for your system.

3. **Login to Prefect and GCP**
   ```bash
   prefect cloud login
   gcloud auth login
   ```

## 🏊‍♂️ Creating a Work Pool

Create a Cloud Run work pool:

```bash
prefect work-pool create --type cloud-run-v2:push --provision-infra my-pool
```

This will set up the necessary infrastructure in your GCP project.

## 🐍 Prepare Your Python Flow

1. Create a file `mypythonscript.py` in `yourproject/flows/`:

   ```python
   from prefect import flow, task

   @task
   def my_task(data):
       return data.upper()

   @flow
   def my_flow(input_data: str):
       result = my_task(input_data)
       print(f"Processed: {result}")

   if __name__ == "__main__":
       my_flow("hello world")
   ```

2. Create a `Dockerfile`:

   ```dockerfile
   FROM prefecthq/prefect:2-python3.10
   COPY requirements.txt .
   RUN pip install -r requirements.txt --trusted-host pypi.python.org --no-cache-dir
   COPY flows /opt/prefect/flows
   CMD ["python", "flows/mypythonscript.py"]
   ```

3. Create a `prefect.yaml` file:

   ```yaml
   name: my-project
   prefect-version: 2.14.13

   build:
   - prefect_docker.deployments.steps.build_docker_image:
       id: build_image
       requires: prefect-docker>=0.3.1
       image_name: gcr.io/your-project/my-flow # this should be provided by the provision-infra command ran before
       tag: latest
       dockerfile: Dockerfile

   push:
   - prefect_docker.deployments.steps.push_docker_image:
       requires: prefect-docker>=0.3.1
       image_name: '{{ build_image.image_name }}'
       tag: '{{ build_image.tag }}'

   pull:
   - prefect.deployments.steps.set_working_directory:
       directory: /opt/prefect

   deployments:
   - name: my-flow-deployment
     entrypoint: flows/mypythonscript.py:my_flow
     work_pool:
       name: my-pool
       job_variables:
         image: '{{ build_image.image }}'
         cpu: 1000m
         memory: 2G
         timeout: 3600
   ```

## 🚀 Deployment

Deploy your flow:

```bash
prefect deploy
```

Select your flow when prompted.

## 🏃‍♂️ Running Your Flow

1. Go to the Prefect Cloud dashboard.
2. Find your deployment under the "Deployments" section.
3. Click "Run" and provide any necessary parameters.
4. Monitor the execution in the Prefect UI and Cloud Run console.

## 🔀 Parallelization (Advanced)

For parallel execution, create a new flow that leverages the Prefect API:

```python
from prefect import flow, task
from prefect.deployments import run_deployment
from prefect.task_runners import ConcurrentTaskRunner

@task
def run_deployment_task(param):
    run_deployment(
        name="my-flow/my-flow-deployment",
        parameters={"input_data": param}
    )

@flow(task_runner=ConcurrentTaskRunner())
def parallel_flow(params):
    for param in params:
        run_deployment_task.submit(param)

if __name__ == "__main__":
    parallel_flow(["data1", "data2", "data3"])
```

Deploy this new flow and run it to execute multiple instances of your original flow in parallel.

## 🎉 Conclusion

You've now set up a Prefect workflow on Cloud Run! Experiment with different configurations and parameters to optimize for your specific use case.

Remember to check the [Prefect documentation](https://docs.prefect.io/) and [Cloud Run documentation](https://cloud.google.com/run/docs) for more advanced features and best practices.

## 📚 Additional Tips and Tricks

For important learned lessons, tips, and tricks, check out our [Prefect Tips and Tricks](prefect_tips_and_tricks.md) guide. It includes information on:
- Cloud Run v1 vs v2
- Overwriting job parameters
- Issues with Prefect Conda image
- And more!

Happy coding! 🚀👨‍💻👩‍💻
