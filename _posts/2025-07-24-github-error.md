---
title: "GitHub Push 실패 문제 해결하기"
excerpt: "포트폴리오 업로드 중 발생한 용량 초과 오류 해결 과정"

categories:
  - Blog
tags:
  - [GitHub, Git, 문제해결, 포트폴리오]

toc: true
toc_sticky: true
 
date: 2025-06-20
last_modified_at: 2025-06-20
---

포트폴리오를 정리해서 GitHub에 올리는 과정에서 파일 용량이 커서 push가 실패하는 문제를 겪었습니다. 이 문제를 해결한 과정을 공유합니다.

## 문제 상황

analytics-projects 디렉토리에 있는 프로젝트를 GitHub에 push하려고 했는데, 다음과 같은 오류가 발생했습니다:

```
error: RPC failed; HTTP 400 curl 22 The requested URL returned error: 400
send-pack: unexpected disconnect while reading sideband packet
fatal: the remote end hung up unexpectedly
```

## 원인 분석

파일 크기를 확인해보니 다음과 같았습니다:

```
$ du -sh *
4.0K    README.md
4.9M    생존 위기에 처한 중소마트를 위한 자구책 모색
```

약 4.9MB 크기의 PDF 파일들이 포함된 폴더 때문에 용량 문제가 발생한 것으로 보였습니다.

## 해결 과정

### 1. 폴더명 변경

먼저 한글 폴더명을 영문으로 변경했습니다:

```
$ mv "생존 위기에 처한 중소마트를 위한 자구책 모색" solution-small-mart
```

### 2. HTTP 버퍼 크기 증가

Git의 HTTP 버퍼 크기를 늘려서 대용량 파일 전송을 가능하게 했습니다:

```
$ git config --global http.postBuffer 524288000
```

이 명령어는 HTTP POST 버퍼를 약 500MB로 설정합니다.

### 3. 재시도

설정 변경 후 다시 push를 시도했습니다:

```
$ git push origin main
```

## 결과

설정 변경 후 성공적으로 push가 완료되었습니다:

```
Writing objects: 100% (7/7), 4.36 MiB | 21.67 MiB/s, done.
Total 7 (delta 0), reused 0 (delta 0), pack-reused 0
To https://github.com/hayeonkimmie/analytics-projects.git
 * [new branch]      main -> main
```

## 추가 팁

앞으로 이런 문제를 예방하기 위한 방법들입니다:

**Git LFS 사용하기**: 대용량 파일은 Git Large File Storage를 활용하면 좋습니다.

```bash
# 1. Git LFS 설치
brew install git-lfs
git lfs install

# 2. PDF 파일 LFS로 트래킹
git lfs track "*.pdf"

# 3. 변경 사항 커밋 & 푸시
git add .gitattributes
git add .
git commit -m "Track PDF files with LFS"
git push origin main
```

**기타 해결책들**:
- **파일 압축**: 필요시 파일을 압축해서 용량 줄이기  
- **파일 분할**: 너무 큰 파일은 여러 개로 나누어 관리
- **gitignore 활용**: 불필요한 대용량 파일은 미리 제외

## 마무리

이번 경험을 통해 Git에서 대용량 파일을 다룰 때의 해결책을 알게 되었습니다. 비슷한 문제를 겪고 계신 분들께 도움이 되길 바랍니다!