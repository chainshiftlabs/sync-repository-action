# Sync Repository Action

다른 레포지토리의 내용을 현재 레포지토리로 동기화하고 자동으로 Pull Request를 생성하는 재사용 가능한 GitHub Action입니다.

## 기능

- 🔄 외부 레포지토리의 특정 브랜치/경로에서 콘텐츠 동기화
- 🚫 설정 가능한 ignore 패턴으로 특정 파일/폴더 제외
- 🔀 자동 PR 생성 및 변경사항 요약
- ⚙️ 다양한 옵션으로 커스터마이징 가능

## 사용 방법

### 기본 사용 예제

```yaml
name: Sync from Upstream

on:
  workflow_dispatch:
  schedule:
    - cron: '0 0 * * 1'  # 매주 월요일 실행

jobs:
  sync:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    
    steps:
      - uses: chainshiftlabs/sync-repository-action@v1
        with:
          source-repo: 'upstream-owner/upstream-repo'
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

### 고급 사용 예제

```yaml
name: Advanced Sync

on:
  workflow_dispatch:

jobs:
  sync:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    
    steps:
      - uses: chainshiftlabs/sync-repository-action@v1
        with:
          source-repo: 'upstream-owner/upstream-repo'
          source-branch: 'develop'
          source-path: 'src'
          target-path: 'external/upstream'
          ignore-patterns: |
            *.test.js
            *.spec.ts
            __tests__/**
            docs/**
            *.md
          pr-title: 'Update from upstream develop branch'
          pr-body: |
            ## Updates from upstream
            
            This PR syncs the latest changes from the upstream repository.
            
            Please review carefully before merging.
          branch-prefix: 'upstream-sync'
          delete-obsolete: 'false'
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `source-repo` | 소스 레포지토리 (형식: owner/repo) | ✅ | - |
| `source-branch` | 소스 브랜치 | ❌ | `main` |
| `source-path` | 동기화할 소스 경로 | ❌ | `` (root) |
| `target-path` | 대상 경로 | ❌ | `` (root) |
| `ignore-patterns` | 제외할 파일/폴더 패턴 (줄바꿈으로 구분) | ❌ | 아래 참조 |
| `pr-title` | Pull Request 제목 | ❌ | `Synchronized {repo}: {latest commit}` |
| `pr-body` | Pull Request 본문 | ❌ | 자동 생성 |
| `branch-prefix` | 동기화 브랜치 접두사 | ❌ | `sync` |
| `commit-message` | 커스텀 커밋 메시지 | ❌ | 자동 생성 |
| `github-token` | GitHub 인증 토큰 (소스와 타겟 레포 모두 접근 가능해야 함) | ✅ | - |
| `create-pr` | PR 생성 여부 | ❌ | `true` |
| `delete-obsolete` | 소스에 없는 파일 삭제 여부 | ❌ | `true` |

## Outputs

| Output | Description |
|--------|-------------|
| `pr-url` | 생성된 Pull Request의 URL |
| `pr-number` | 생성된 Pull Request 번호 |
| `has-changes` | 변경사항 감지 여부 (true/false) |
| `branch-name` | 생성된 브랜치 이름 |

## 기본 Ignore 패턴

기본적으로 다음 패턴이 제외됩니다:

- `*.Dockerfile`, `Dockerfile.*` - Docker 관련 파일
- `package/**` - package 디렉토리
- `node_modules/**` - Node.js 의존성
- `.git/**` - Git 메타데이터
- `.github/**` - GitHub 설정
- `*.log`, `*.tmp` - 로그 및 임시 파일
- `.env`, `.env.*` - 환경 변수 파일
- `dist/**`, `build/**` - 빌드 출력
- `coverage/**` - 테스트 커버리지
- `.DS_Store`, `Thumbs.db` - 시스템 파일

## 사용 시나리오

### 1. 정기적인 업스트림 동기화

```yaml
- uses: chainshiftlabs/sync-repository-action@v1
  with:
    source-repo: 'facebook/react'
    source-path: 'packages/react/src'
    target-path: 'vendor/react'
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### 2. 문서만 동기화

```yaml
- uses: chainshiftlabs/sync-repository-action@v1
  with:
    source-repo: 'company/documentation'
    source-path: 'docs'
    target-path: 'docs/external'
    ignore-patterns: |
      *.test.*
      examples/**
      drafts/**
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### 3. PR 없이 직접 커밋

```yaml
- uses: chainshiftlabs/sync-repository-action@v1
  with:
    source-repo: 'team/shared-configs'
    create-pr: 'false'
    commit-message: 'Update shared configurations'
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### 4. Private 레포지토리에서 동기화

```yaml
- uses: chainshiftlabs/sync-repository-action@v1
  with:
    source-repo: 'company/private-repo'
    github-token: ${{ secrets.PAT_WITH_REPO_ACCESS }}  # 두 레포 모두 접근 가능한 토큰
```

Private 레포지토리를 동기화할 때는 소스와 타겟 레포지토리 모두에 접근 권한이 있는 Personal Access Token (PAT)을 제공해야 합니다.

## 권한 요구사항

워크플로우에서 다음 권한이 필요합니다:

```yaml
permissions:
  contents: write       # 파일 수정 및 브랜치 생성
  pull-requests: write  # PR 생성 (create-pr가 true인 경우)
```

## 주의사항

- 동기화는 기존 파일을 덮어쓸 수 있으므로 중요한 변경사항은 백업하세요
- `delete-obsolete: true`일 경우 소스에 없는 파일이 삭제됩니다
- Private 레포지토리에서 동기화할 때는 적절한 권한의 토큰이 필요합니다

### Private 레포지토리 설정

Private 레포지토리에서 동기화하려면:

1. **Personal Access Token (PAT) 생성**:
   - GitHub Settings → Developer settings → Personal access tokens
   - `repo` 스코프 선택 (소스와 타겟 레포 모두에 대한 권한 필요)
   
2. **Secret 추가**:
   - 레포지토리 Settings → Secrets and variables → Actions
   - 새 secret 추가 (예: `PAT_WITH_REPO_ACCESS`)
   
3. **워크플로우에서 사용**:
   ```yaml
   github-token: ${{ secrets.PAT_WITH_REPO_ACCESS }}
   ```

## 라이선스

MIT License