# Base: Amazon Linux 2023 (same OS as aws/codebuild/amazonlinux2-x86_64-standard:5.0)
# The CodeBuild managed image is not publicly pullable — it's only available
# as a CodeBuild environment. We use the official AL2023 base and install
# exactly what the pre_build phase downloads.
FROM public.ecr.aws/amazonlinux/amazonlinux:2023

ARG HELM_VERSION=3.16.4
ARG CODER_VERSION=2.30.2

# The AL2023 minimal image ships curl-minimal which conflicts with
# the full curl package. Swap it out first, then install everything.
RUN dnf swap -y curl-minimal curl \
    && dnf install -y \
    python3.12 \
    python3.12-pip \
    git \
    unzip \
    jq \
    tar \
    gzip \
    && dnf clean all \
    && ln -sf /usr/bin/python3.12 /usr/bin/python3 \
    && ln -sf /usr/bin/pip3.12 /usr/bin/pip3

# --- Mirror pre_build commands from coder_deployment.yaml ---

# 1. pip3 install --upgrade --user awscli
RUN pip3 install --upgrade --user awscli \
    && echo 'export PATH=$HOME/.local/bin:$PATH' >> /etc/profile.d/awscli.sh
ENV PATH="/root/.local/bin:${PATH}"

# 2. kubectl (latest stable)
RUN curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl" \
    && chmod +x ./kubectl \
    && mv ./kubectl /usr/local/bin/kubectl

# 3. eksctl (latest, using uname -s like the original)
RUN curl --silent --fail --retry 5 --location \
      "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" \
      | tar xz -C /tmp \
    && mv -v /tmp/eksctl /usr/local/bin \
    && eksctl version

# 4. helm v3.16.4
RUN curl --silent --fail --retry 5 --location \
      "https://get.helm.sh/helm-v${HELM_VERSION}-linux-amd64.tar.gz" \
      | tar xz -C /tmp \
    && mv -v /tmp/linux-amd64/helm /usr/local/bin \
    && rm -rf /tmp/linux-amd64 \
    && helm version

# 5. coder CLI
RUN curl --silent --fail --retry 5 -L https://coder.com/install.sh \
      | sh -s -- --version ${CODER_VERSION} --method standalone \
    && coder --version

# 6. terraform (latest)
RUN TERRAFORM_VERSION=$(curl --silent --fail --retry 5 --location \
      "https://api.github.com/repos/hashicorp/terraform/releases/latest" \
      | grep tag_name | cut -d '"' -f 4 | sed 's/v//') \
    && curl --silent --fail --retry 5 --location \
      "https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip" \
      -o terraform.zip \
    && unzip terraform.zip \
    && mv terraform /usr/local/bin/ \
    && chmod +x /usr/local/bin/terraform \
    && rm terraform.zip \
    && terraform version

# Verify all installs
RUN echo "=== Installed versions ===" \
    && aws --version \
    && kubectl version --client \
    && eksctl version \
    && helm version --short \
    && terraform version \
    && coder --version \
    && python3 --version \
    && git --version

ENTRYPOINT ["/bin/bash", "-c"]
