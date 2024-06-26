<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css"
        integrity="sha384-T3c6CoIi6uLrA9TneNEoa7RxnatzjcDSCmG1MXxSR1GAsXEV/Dwwykc2MPK8M2HN" crossorigin="anonymous">
</head>

<body>
    <div class="container mt-5">
        <div class="jumbotron">
            <h1 class="display-4">WebDeployer</h1>
            <p class="lead">WebDeployer automates the deployment of static websites to AWS EC2 instances using Docker
                and GitHub Actions. With a focus on simplicity and efficiency, WebDeployer leverages self-hosted runners
                to provide a secure and robust CI/CD pipeline for web applications.</p>
            <h2>Prerequisites</h2>
            <ul>
            <li><strong>GitHub Repository:</strong> Ensure you have a GitHub repository where you want to deploy the website.</li>
            <li><strong>EC2 Instance:</strong> An EC2 instance with Docker installed.</li>
            <li><strong>Self-Hosted Runner:</strong> Set up on the EC2 instance.</li>
            </ul>
   <h2>Step 1: Set Up Your EC2 Instance</h2>
    <h3>Launch an EC2 Instance:</h3>
    <ol>
        <li>Log in to the AWS Management Console.</li>
        <li>Navigate to the EC2 dashboard and click "Launch Instance".</li>
        <li>Choose an Amazon Machine Image (AMI) (e.g., Ubuntu).</li>
        <li>Select an instance type (e.g., t2.micro for free tier).</li>
        <li>Configure instance details, including setting up security groups to allow SSH (port 22) and HTTP (port 80) access.</li>
        <li>Launch the instance and note the instance ID, public DNS, and private IP address.</li>
    </ol>
  <h3>Connect to Your EC2 Instance:</h3>
    <pre><code>ssh -i /path/to/your-key.pem ec2-user@your-ec2-instance-public-dns</code></pre>
<h3>Install Docker:</h3>
    <pre><code>sudo apt update -y
sudo apt install docker -y
sudo service docker start
sudo usermod -aG docker ec2-user</code></pre>

  <h2>Step 2: Configure the Self-Hosted Runner</h2>
    <h3>Register the Runner with GitHub:</h3>
    <ol>
        <li>Go to your GitHub repository.</li>
        <li>Click on the "Settings" tab.</li>
        <li>In the sidebar, click on "Actions" and then "Runners".</li>
        <li>Click the "New self-hosted runner" button.</li>
        <li>Follow the instructions to download and configure the runner application on your EC2 instance.</li>
    </ol>

  <h2>Step 3: Create a GitHub Actions Workflow</h2>
    <h3>Create a Workflow File:</h3>
    <p>In your repository, create a directory: <code>.github/workflows</code>.</p>
    <p>Create a file named <code>deploy.yml</code> in this directory.</p>

   <h3>Define the Workflow:</h3>
   
    name: Deploy

    on: [push]

    jobs:
     deploy:
      runs-on: self-hosted

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Check Current Directory
      run: pwd

    - name: Verify Docker Installation
      run: |
        sudo docker --version
        sudo systemctl status docker

    - name: Check Running Docker Services
      run: sudo docker ps -a

    - name: Stop Docker Container
      run: sudo docker stop csn-nginx || true

    - name: Delete the Docker Container
      run: sudo docker rm csn-nginx || true

    - name: Build Nginx Docker Image
      run: sudo docker build -t csn-nginx .

    - name: Run the Docker Container
      run: sudo docker run -d --name csn-nginx -p 8080:80 csn-nginx

    - name: Check Running Docker Services After Deployment
      run: sudo docker ps -a

    - name: Check Docker Container Logs
      run: sudo docker logs csn-nginx

    - name: Test the Web Service
      run: |
        echo "Waiting for the service to start..."
        sleep 10  # Wait for 10 seconds to give the service time to start
        curl -v http://localhost:8080 || (echo "Curl failed"; sudo docker logs csn-nginx)

  <h2>Step 4: Trigger the Pipeline</h2>
    <h3>Push Code Changes:</h3>
    <p>Push changes to the main branch of your repository. This will trigger the GitHub Actions workflow.</p>

  <h3>Monitor the Workflow:</h3>
    <p>Go to the "Actions" tab in your GitHub repository to monitor the progress of your workflow.</p>

  <h2>Summary</h2>
    <p>This setup leverages GitHub Actions to deploy a website to an EC2 instance using a self-hosted runner. The workflow checks out your code, builds a Docker image, and runs a Docker container to serve your website via Nginx. By using a self-hosted runner, the deployment is executed directly on your EC2 instance, ensuring a secure and efficient workflow.</p>
<p>By following these steps, you can securely and efficiently deploy your website to an EC2 instance using GitHub Actions with a self-hosted runner.</p>

