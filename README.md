# github-actions

GGQ common reusable workflows

## CreateIssueFromTodo

TODO 주석을 찾아 Issue로 변환해주는 Workflow

### Inputs

| name       | description          | required | example                                   |
| ---------- | -------------------- | -------- | ----------------------------------------- |
| repository | Caller's repository  | Y        | `ggqcompany/back-node-master-auth-server` |
| ref_name   | Caller's branch name | Y        | `develop`                                 |

### Secrets

| name                   | description                                |
| ---------------------- | ------------------------------------------ |
| TODO_ACTIONS_MONGO_URL | TODO 주석 reference 체크를 위한 몽고DB URL |

### Caller's example

```
name: CreateIssueFromTodo

on:
  push:
    branches:
      - "develop"

jobs:
  CreateIssueFromTodo:
    uses: ggqcompany/github-actions/.github/workflows/CreateIssueFromTodo.yml@main
    secrets: inherit
    with:
      repository: ${{ github.repository }}
      ref_name: ${{ github.ref_name }}

```

## DeployToDev, DeployToStg, DeployToProd

- DeployToDev : DEV 환경으로 배포, `feature`, `develop` 브랜치만 허용
- DeployToStg : STG 환경으로 배포, `release`, `hotfix` 브랜치만 허용
- DeployToProd : PROD 환경으로 배포, `main` 브랜치만 허용

### Inputs

| name                   | description                                                                                                                  | example                                    |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| repository             | Caller's repository                                                                                                          | `ggqcompany/back-node-master-auth-server`  |
| ref_name               | Caller's branch name                                                                                                         | `release/1.0.0+123456`                     |
| commit                 | 배포하고자 하는 commit                                                                                                       | `9f6be1bf4a6d019d60c376c8e0c7f0b9e7e294c3` |
| environment            | 배포하고자 하는 환경                                                                                                         | `DEV`, `STG`, `PROD`                       |
| region                 | 배포하고자 하는 리전                                                                                                         | `ap-northeast-2`                           |
| account_id_dev         | AWS account id                                                                                                               | `12345678901`                              |
| account_id_stg         | AWS account id                                                                                                               | `12345678901`                              |
| account_id_prod        | AWS account id                                                                                                               | `12345678901`                              |
| product_code           | Product Code ([Naming Convention 참고](https://docs.google.com/document/d/1loswue7bJ2u8h6eDSx0KFk6fhknEel5QKn-Tb96tonQ))     | `ggq`                                      |
| application_code       | Application Code ([Naming Convention 참고](https://docs.google.com/document/d/1loswue7bJ2u8h6eDSx0KFk6fhknEel5QKn-Tb96tonQ)) | `lol_auth`                                 |
| s3_api_doc_folder_name | S3 API 문서 bucket 내 doc 폴더 위치                                                                                          | `s3_api_doc_folder_name`                   |
| server_name            | Slack 노티 시 보여 줄 이름                                                                                                   | `Auth Server`                              |

### Secrets

| name                  | description                           |
| --------------------- | ------------------------------------- |
| AWS_ACCESS_KEY_ID     | AWS_ACCESS_KEY_ID                     |
| AWS_SECRET_ACCESS_KEY | AWS_SECRET_ACCESS_KEY                 |
| SLACK_WEBHOOK_URL     | 배포 시 Slack 노티를 위한 webhook URL |

### Caller's example

```
name: ManualDeployToDev

# feature 브랜치에서 DEV 환경에 배포 필요 시 수동으로 Workflow를 수행한다.
# 대상 feature 브랜치의 최신 commit으로 배포

on:
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        required: true
        description: 배포 환경 (수정 불가)
        options:
          - DEV
      region:
        type: choice
        required: true
        description: AWS region
        options:
          # DEV환경은 [ap-northeast-2] 로 고정
          - ap-northeast-2

jobs:
  DeployToDev:
    uses: ggqcompany/github-actions/.github/workflows/DeployToDev.yml@main
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID_DEV }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY_DEV }}
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
    with:
      # Checkout 시 필요한 Input Data
      repository: ${{ github.repository }}
      ref_name: ${{ github.ref_name }}
      # AWS 배포 시 필요한 데이터
      commit: ${{ github.sha }}
      environment: ${{ inputs.environment }}
      region: ${{ inputs.region }}
      account_id_dev: "12345678901"
      account_id_stg: "12345678901"
      account_id_prod: "12345678901"
      product_code: ggq
      application_code: lol_auth
      s3_api_doc_folder_name: ggq-lol-auth
      # Slack 송신에 필요한 데이터
      server_name: Auth Server
```

## RollbackDeploy

태깅 된 버전으로 재배포 시 사용

### Inputs

| name                   | description                                                                                                                  | example                                    |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| tag                    | 배포하고자 하는 태그 이름                                                                                                    | `1.0.0-rc.1+123456`                        |
| repository             | Caller's repository                                                                                                          | `ggqcompany/back-node-master-auth-server`  |
| ref_name               | Caller's branch name                                                                                                         | `release/1.0.0+123456`                     |
| commit                 | 배포하고자 하는 commit                                                                                                       | `9f6be1bf4a6d019d60c376c8e0c7f0b9e7e294c3` |
| environment            | 배포하고자 하는 환경                                                                                                         | `DEV`, `STG`, `PROD`                       |
| region                 | 배포하고자 하는 리전                                                                                                         | `ap-northeast-2`                           |
| account_id_dev         | AWS account id                                                                                                               | `12345678901`                              |
| account_id_stg         | AWS account id                                                                                                               | `12345678901`                              |
| account_id_prod        | AWS account id                                                                                                               | `12345678901`                              |
| product_code           | Product Code ([Naming Convention 참고](https://docs.google.com/document/d/1loswue7bJ2u8h6eDSx0KFk6fhknEel5QKn-Tb96tonQ))     | `ggq`                                      |
| application_code       | Application Code ([Naming Convention 참고](https://docs.google.com/document/d/1loswue7bJ2u8h6eDSx0KFk6fhknEel5QKn-Tb96tonQ)) | `lol_auth`                                 |
| s3_api_doc_folder_name | S3 API 문서 bucket 내 doc 폴더 위치                                                                                          | `s3_api_doc_folder_name`                   |
| server_name            | Slack 노티 시 보여 줄 이름                                                                                                   | `Auth Server`                              |

### Secrets

| name                       | description                           |
| -------------------------- | ------------------------------------- |
| AWS_ACCESS_KEY_ID_STG      | AWS_ACCESS_KEY_ID for STG             |
| AWS_SECRET_ACCESS_KEY_STG  | AWS_SECRET_ACCESS_KEY for STG         |
| AWS_ACCESS_KEY_ID_PROD     | AWS_ACCESS_KEY_ID for PROD            |
| AWS_SECRET_ACCESS_KEY_PROD | AWS_SECRET_ACCESS_KEY for PROD        |
| SLACK_WEBHOOK_URL          | 배포 시 Slack 노티를 위한 webhook URL |

### Caller's example

```
name: RollbackDeploy

# 태그이름으로 Rollback 배포 진행
# rc 버전 태그일 경우 STG 배포
# rc 버전 태그가 아닐 경우 PROD 배포

on:
  workflow_dispatch:
    inputs:
      tag:
        type: string
        required: true
        description: 배포할 tag 이름을 입력하세요.
      region:
        type: choice
        required: true
        description: 배포할 AWS region를 선택하세요.
        options:
          - ap-northeast-2

jobs:
  RollbackDeploy:
    uses: ggqcompany/github-actions/.github/workflows/RollbackDeploy.yml@main
    secrets: inherit
    with:
      tag: ${{ inputs.tag }}
      # Checkout 시 필요한 Input Data
      repository: ${{ github.repository }}
      ref_name: ${{ github.ref_name }}
      # AWS 배포 시 필요한 데이터
      commit: ${{ github.sha }}
      region: ${{ inputs.region }}
      account_id_dev: "12345678901"
      account_id_stg: "12345678901"
      account_id_prod: "12345678901"
      product_code: ggq
      application_code: lol_auth
      s3_api_doc_folder_name: ggq-lol-auth
      # Slack 송신에 필요한 데이터
      server_name: Auth Server

```

## ReportIssue

Open 되어있는 Issue를 Slack으로 알려주는 Workflow

### Inputs

| name       | description          | required | example                                   |
| ---------- | -------------------- | -------- | ----------------------------------------- |
| repository | Caller's repository  | Y        | `ggqcompany/back-node-master-auth-server` |
| ref_name   | Caller's branch name | Y        | `develop`                                 |
| milestone  | Target milestone     | N        | `1.0.0+123456`                            |

### Secrets

| name                   | description                                 |
| ---------------------- | ------------------------------------------- |
| SLACK_TODO_WEBHOOK_URL | Open 된 Issue 노티를 위한 Slack webhook URL |

### Caller's example

```
name: ReportIssue

# cron: https://crontab.guru/
# 이 크론은 매주 월요일 한국 시간 9시 30분(AM)에 실행됩니다.
on:
  schedule:
    - cron: 30 0 * * 1

jobs:
  ReportIssue:
    uses: ggqcompany/github-actions/.github/workflows/ReportIssue.yml@main
    secrets: inherit
    with:
      repository: ${{ github.repository }}
      ref_name: ${{ github.ref_name }}
```

## SetTagForStg, SetTagForProd

- SetTagForStg : `release`, `hotfix` branch에 rc버전 태깅
- SetTagForProd : `release` branch -> `main` branch 로 머지 후 `main` 브랜치에 PROD 버전 태깅

### Inputs

| name       | description          | required | example                                   |
| ---------- | -------------------- | -------- | ----------------------------------------- |
| repository | Caller's repository  | Y        | `ggqcompany/back-node-master-auth-server` |
| ref_name   | Caller's branch name | Y        | `develop`                                 |

### Secrets

| name                   | description                                 |
| ---------------------- | ------------------------------------------- |
| SLACK_TODO_WEBHOOK_URL | Open 된 Issue 노티를 위한 Slack webhook URL |

### Caller's example

```
on:
  workflow_dispatch:

jobs:
  SetTagForStg:
    uses: ggqcompany/github-actions/.github/workflows/SetTagForStg.yml@main
    secrets: inherit
    with:
      repository: ${{ github.repository }}
      ref_name: ${{ github.ref_name }}
```
