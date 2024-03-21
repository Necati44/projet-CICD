# Define a base stage that uses the official python runtime base image
FROM python:3.8-slim

# Add curl for healthcheck
RUN apt-get update && \
    apt-get install -y --no-install-recommends && \
    rm -rf /var/lib/apt/lists/*

# Set the application directory
WORKDIR /usr/local/app

# Install our requirements.txt
COPY requirements.txt ./requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Copy our code from the current folder to the working directory inside the container
COPY . .

# Make port 3001 available for links and/or publish
EXPOSE 3001

# Define our command to be run when launching the container
CMD ["python3", "app.py"]