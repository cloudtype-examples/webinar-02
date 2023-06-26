<br/>
<br/>

<p align="center">
<img src="https://files.cloudtype.io/logo/cloudtype-logo-horizontal-black.png" width="50%" alt="Cloudtype"/>
</p>

<br/>
<br/>

# 클라우드타입 웨비나 #02 <br/> GitHub Actions, Private Registry를 활용한<br/>CI/CD 파이프라인 구축하기 <!-- omit in toc -->

## 목차 <!-- omit in toc -->

- [🛠️ 준비사항](#️-준비사항)
- [🗄️ Private Registry](#️-private-registry)
  - [AWS Elastic Registry](#aws-elastic-registry)
    - [토큰 생성 명령어](#토큰-생성-명령어)
    - [GitHub Actions Workflow](#github-actions-workflow)
    - [GitHub Repository Secrets](#github-repository-secrets)
  - [GCP Artifact Registry](#gcp-artifact-registry)
    - [GitHub Actions Workflow](#github-actions-workflow-1)
    - [GitHub Repository Secrets](#github-repository-secrets-1)
- [📖 References](#-references)
- [💬 Contact](#-contact)

## 🛠️ 준비사항

- [GitHub 계정](https://github.com/)
- [클라우드타입 계정](https://cloudtype.io/)
- [AWS 계정](https://aws.amazon.com/ko)
- [Google Cloud Platform 계정](https://cloud.google.com/?hl=ko)
- [예제 소스 GitHub 저장소](https://github.com/cloudtype-examples/webinar-02)

## 🗄️ Private Registry

### AWS Elastic Registry

#### 토큰 생성 명령어

- AWS CLI 설치

  ```yaml
    curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
    sudo installer -pkg AWSCLIV2.pkg -target /
  ```

- AWS 계정 설정
  
  ```yaml
    aws configure
  ```

- AWS 토큰 발급

  ```yaml
    aws ecr get-login-password --region ap-northeast-2

  ```

#### GitHub Actions Workflow

```yaml
name: Create and publish a Docker image to AWS ECR, Deploy to Cloudtype

on:
  push:
    branches:
      - main

env:
  REGISTRY: ${{ secrets.AWS_ECR_REGISTRY }}
  IMAGE_NAME: [이미지명]

jobs:
  build-and-push-image:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Log in to the Container registry
        uses: docker/login-action@65b78e6e13532edd9afa3aa52ac7964289d1a9c1
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ secrets.AWS_ACCESS_KEY_ID }}
          password: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

      - name: Extract metadata (tags, labels) for Docker
        id: meta
        uses: docker/metadata-action@9ec57ed1fcdbf14dcef7dfbe97b2010124a938b7
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=sha
      - name: Build and push Docker image
        uses: docker/build-push-action@f2a1d5e99d037542a71f64918e516c093c6f3fc4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}

      - name: Deploy to Cloudtype
        uses: cloudtype-github-actions/deploy@v1
        with:
          token: ${{ secrets.CLOUDTYPE_TOKEN }}
          project: [스페이스명]/[프로젝트명]
          stage: main
          yaml: |
            name: [서비스명]
            app: container
            options:
              ports: [포트번호]
              image: ${{ steps.meta.outputs.tags }}
```

#### GitHub Repository Secrets

- `AWS_ECR_REGISTRY`: AWS ECR Registry 주소(리포지토리 경로 제외)
- `AWS_ACCESS_KEY_ID`: AWS 액세스 키 ID
- `AWS_SECRET_ACCESS_KEY`: AWS 시크릿 액세스 키
- `CLOUDTYPE_TOKEN`: 클라우드타입 API 키

---

### GCP Artifact Registry

#### GitHub Actions Workflow

```yaml
name: Create and publish a Docker image to GCP Artifact Registry, Deploy to Cloudtype

on:
  push:
    branches:
      - main

env:
  REGISTRY: ${{ secrets.GAR_REGISTRY }}
  IMAGE_NAME: [이미지명]

jobs:
  build-and-push-image:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3
      
      - name: Log in to the Container registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: _json_key
          password: ${{ secrets.GAR_JSON_KEY }}
          
      - name: Extract metadata (tags, labels) for Docker
        id: meta
        uses: docker/metadata-action@9ec57ed1fcdbf14dcef7dfbe97b2010124a938b7
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=sha
      - name: Build and push Docker image
        uses: docker/build-push-action@f2a1d5e99d037542a71f64918e516c093c6f3fc4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}

      - name: Deploy to Cloudtype
        uses: cloudtype-github-actions/deploy@v1
        with:
          token: ${{ secrets.CLOUDTYPE_TOKEN }}
          project: [스페이스명]/[프로젝트명]
          stage: main
          yaml: |
            name: [서비스명]
            app: container
            options:
              ports: [포트번호]
              image: ${{ steps.meta.outputs.tags }}
```

#### GitHub Repository Secrets

- `GAR_REGISTRY`: Artifact Registry 주소
- `GAR_JSON_KEY`: Artifact Registry 서비스 계정 JSON KEY
- `CLOUDTYPE_TOKEN`: 클라우드타입 API 키

## 📖 References

- [클라우드타입 Docs](https://docs.cloudtype.io/)

- [클라우드타입 FAQ](https://help.cloudtype.io/guide/faq)
  
- [GitHub Actions Docs](https://docs.github.com/en/actions)

- [AWS Elastic Container Registry](https://docs.aws.amazon.com/ko_kr/ecr/index.html)

- [GCP Artifact Registry](https://cloud.google.com/artifact-registry/docs?hl=ko)

## 💬 Contact

- [Discord](https://discord.gg/U7HX4BA6hu)
