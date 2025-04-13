# Docker Image Signing and CI/CD Trigger Guide

This guide outlines the steps to tag a Docker image, sign it with Docker Content Trust, manage keys, and trigger a pubgrade using a curl command.

## Prerequisites
- Docker installed and configured.
- Access to the Docker registry (e.g., Docker Hub).
- A GitHub repository with secrets configured.
- Access to the CI/CD system (e.g., Pubgrade).
- Docker Content Trust keys generated in `~/.docker/trust/private/`.

## Steps

### 1. Tag the Docker Image
Tag the Docker image to prepare it for signing and pushing.

```bash
docker tag image:tag repo/image:tag
```

### 2. Sign the Docker Image
Sign the tagged image using Docker Content Trust to ensure its integrity.

```bash
docker trust sign repo/image:tag
```

You will be prompted to enter passphrases for the following keys:
- **Root key** (e.g., ID: `261edfc`)
- **Repository key** (e.g., ID: `a4f3043`)
- **Signer key** (e.g., ID: `8cbb14c`)

Upon successful signing, the output will confirm:
- Creation of the signer (`akash7778`).
- Initialization of the signed repository.
- Pushing of trust data to the registry (e.g., `docker.io/repo/image:tag`).

Example output:
```
The push refers to repository [docker.io/repo/image]
...
0.1: digest: sha256:19b32dc5db8fe202dba579657a2323122b043f39a10b1f948f60a7df8fa5782e size: 2634
Successfully signed docker.io/repo/image:tag
```

### 3. Copy Keys to GitHub Secrets
Locate the signer key (e.g., `8cbb14c`) in the `~/.docker/trust/private/` directory.

1. Navigate to the directory:
   ```bash
   cd ~/.docker/trust/private/
   ```
2. Identify the key file (e.g., `8cbb14c.key`).
3. Copy the key content or ID.
4. Add the key to your GitHub repository secrets:
   - Go to your repository on GitHub.
   - Navigate to **Settings** > **Secrets and variables** > **Actions** > **Secrets**.
   - Create a new secret (e.g., `DOCKER_SIGNER_KEY`) and paste the key content or ID.

### 4. Trigger Pubgrade
Use a `curl` command to trigger the Pubgrade for building and deploying the image.

```bash
curl --location 'https://pubgrade.dyn.cloud.e-infra.cz/repositories/cwaitc/builds' \
--header 'X-Project-Access-Token: crstoor.glnw.i-s---o.roawlnwnsci' \
--header 'Content-Type: application/json' \
--data '{
    "images": [
        {
            "name": "repo/image:tag",
            "location": "./Dockerfile"
        }
    ],
    "head_commit": {
        "branch": "main"
    },
    "dockerhub_token": "<token>"
}'
```

Replace `<token>` with your Docker Hub token, which should also be stored as a GitHub secret for security.

## Notes
- Ensure passphrases for keys are securely stored and not shared.
- Verify the CI/CD endpoint (`https://pubgrade.dyn.cloud.e-infra.cz`) is accessible and the project access token is valid.
- The `dockerhub_token` should be retrieved from a secure source, such as GitHub secrets, rather than hardcoded.
- Always validate the image digest after signing to ensure integrity.

## Troubleshooting
- If the `docker trust sign` command fails, ensure your keys are correctly set up in `~/.docker/trust/private/`.
- For CI/CD failures, check the response from the `curl` command and verify the endpoint, token, and payload.
