# Kubernetes Workspace with Kiro CLI

This Coder template deploys a Kubernetes-based development workspace with the Kiro CLI pre-installed, along with AWS utilities for cloud development.

## Features

- **Kiro CLI**: AI-powered development assistant accessible via command line
- **AWS CLI**: Command-line interface for AWS services
- **AWS CDK**: Infrastructure as Code toolkit for AWS
- **code-server**: VS Code in the browser
- **Kiro Web UI**: Browser-based Kiro interface
- **Persistent Storage**: Home directory persists across workspace restarts

## What's Included

### Pre-installed Tools
- Kiro CLI (latest version)
- AWS CLI v2
- AWS CDK
- Node.js 20.x LTS
- npm 11.3.0

### Coder Apps
- **code-server**: Web-based VS Code IDE
- **Kiro**: Browser-based Kiro interface
- **Authenticate Kiro**: One-click button to authenticate Kiro CLI
- **Preview**: Preview web applications running on localhost:3000

## Getting Started

1. Create a workspace from this template
2. Once the workspace starts, click the "Authenticate Kiro" button to log in to Kiro
3. Open code-server or Kiro to start developing
4. Use the terminal to run `kiro` commands

## Kiro CLI Usage

After authenticating, you can use Kiro from the terminal:

```bash
# Start a chat session
kiro chat

# Ask Kiro a question
kiro ask "How do I deploy to AWS?"

# Get help
kiro --help
```

## Configuration

### Parameters
- **CPU cores**: 2-8 cores (default: 4)
- **Memory**: 4-16 GB (default: 8 GB)
- **PVC storage size**: 10-50 GB (default: 30 GB)

### Namespace
The default Kubernetes namespace is `coder`. Update the `namespace` variable if deploying to a different namespace.

## AWS Integration

This template includes AWS CLI and CDK for seamless AWS development. Ensure your workspace has appropriate IAM permissions configured via the Kubernetes service account.

## Notes

- The workspace uses the `codercom/enterprise-base:ubuntu` image
- All tools are installed during the first workspace startup
- Subsequent starts will skip installation if tools are already present
- The home directory (`/home/coder`) persists across workspace restarts
