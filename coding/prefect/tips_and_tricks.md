# 🧠 Prefect Tips and Tricks

This document contains important lessons learned, tips, and tricks for using Prefect with Google Cloud Run. These insights will help you avoid common pitfalls and optimize your workflows.

## 🔄 Cloud Run v1 vs v2

### Cloud Run v1
- **Running time limit**: 1 hour (3600 seconds)
- **Use case**: Suitable for shorter-running tasks

### Cloud Run v2
- **Running time limit**: 24 hours (86400 seconds)
- **Use case**: Ideal for longer-running processes
- **Note**: As of writing, this is considered experimental

To specify the version in your work pool creation:

```bash
# For v1
prefect work-pool create --type cloud-run:push --provision-infra my-pool-v1

# For v2
prefect work-pool create --type cloud-run-v2:push --provision-infra my-pool-v2
```

## 🔧 Overwriting Job Parameters

You can overwrite job parameters in your `prefect.yaml` file under the `work_pool` section:

```yaml
work_pool:
  name: my-pool
  job_variables:
    image: '{{ build_image.image }}'
    cpu: 2000m  # Overwritten CPU allocation
    memory: 4G  # Overwritten memory allocation
    timeout: 7200  # Overwritten timeout in seconds
```

This allows you to customize resource allocation for specific deployments.
## 🐳 Docker and Conda Environment Issues

When working with Prefect, Conda environments, and Docker, you might encounter some challenges. Here are some insights and solutions based on real-world experience:

### Conda Environment in Docker

While using the Prefect Conda image seemed like a good solution, it can cause issues when Prefect tries to run the flow in the Docker container. The following Dockerfile configuration was attempted but didn't work with Cloud Run push with infra-provide:

```dockerfile
FROM prefecthq/prefect:2-python3.10-conda

COPY environment.yml .
COPY pyproject.toml .
COPY poetry.lock .
COPY requirements.txt .
COPY setup.cfg .
COPY setup.py .
COPY xmatics .

RUN apt-get update && apt-get install -y libarchive13
RUN conda install -c conda-forge mamba
RUN mamba env update --prefix /opt/conda/envs/prefect -f environment.yml
RUN conda activate prefect

ENV PYTHONUNBUFFERED True

COPY flows/ /opt/prefect/flows/

ENTRYPOINT ["/bin/bash", "--login", "-c", "prefect agent start -q cloudrun"]
```

This configuration worked with the old way of releasing when an always-online worker needed to be deployed on a VM in GCP, but it's not compatible with the current Cloud Run push setup.

### Working Solution

The current working solution involves using a Conda environment with Mamba and Poetry installed locally, and then using Poetry to build a distribution of your package. Here's the recommended approach:

1. Create an `environment.yml` file to manage difficult-to-install libraries like GDAL and Fiona:

```yaml
name: prefect-env
channels:
  - conda-forge
  - defaults
dependencies:
  - python=3.10
  - gdal
  - fiona
  - mamba
  - poetry
  # Add other conda-managed dependencies here
```

2. Use Poetry to manage other dependencies in your `pyproject.toml` file.

3. Build your package using Poetry:

```bash
poetry build
```

4. Create a Dockerfile for Prefect that uses the built package:

```dockerfile
FROM prefecthq/prefect:2.14.17-python3.10

# Add our requirements.txt file to the image and install dependencies
COPY requirements.txt .

RUN apt-get update && apt-get install -y libarchive13 libgdal-dev

RUN pip install GDAL==$(gdal-config --version) --global-option=build_ext --global-option="-I/usr/include/gdal"
RUN pip install -r requirements.txt --trusted-host pypi.python.org --no-cache-dir

COPY dist/myproject-0.1.0.tar.gz .
RUN pip install myproject-0.1.0.tar.gz

COPY setup.py .
COPY setup.cfg .

COPY myproject myproject
# Add our flow code to the image
COPY myproject/flows /opt/prefect/flows

RUN pip install .

CMD ["python", "/opt/prefect/flows/main_flow.py"]
```

This approach allows you to:
- Manage complex dependencies like GDAL and Fiona using Conda/Mamba
- Use Poetry for other Python dependencies
- Create a distributable package of your project
- Use a Prefect-based Docker image that can install and run your package

Remember to adjust version numbers and paths according to your specific project structure.


## 💡 Additional Tips

1. **Logging**: Use Prefect's built-in logging for better visibility:
   ```python
   from prefect import get_run_logger

   @task
   def my_task():
       logger = get_run_logger()
       logger.info("This will appear in Prefect logs")
   ```

2. **Retries**: Implement retries for more robust tasks:
   ```python
   from prefect import task

   @task(retries=3, retry_delay_seconds=60)
   def my_flaky_task():
       # Your code here
   ```

3. **Caching**: Use caching to speed up repetitive computations:
   ```python
   from prefect import task
   from prefect.tasks.core.constants import THREAD_CACHE

   @task(cache_key_fn=lambda: "my-cache-key", cache_expiration=timedelta(hours=1))
   def my_expensive_task():
       # Your code here
   ```

4. **Secrets Management**: Use Prefect Blocks for managing secrets securely:
   ```python
   from prefect.blocks.system import Secret

   secret_block = Secret.load("my-secret-block")
   secret_value = secret_block.get()
   ```

Remember to keep your Prefect and Cloud Run configurations up to date, and regularly review the official documentation for new features and best practices.

Happy flow building! 🌊🏗️