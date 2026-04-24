# QMD 한글 사용자 매뉴얼

> **QMD (Query Markup Documents)** — 온디바이스 하이브리드 검색 엔진  
> BM25 전문 검색 + 벡터 시맨틱 검색 + LLM 재순위화를 로컬에서 모두 실행합니다.

---

## 목차

1. [시스템 요구사항](#1-시스템-요구사항)
2. [설치](#2-설치)
   - [macOS](#21-macos)
   - [Windows (CMD)](#22-windows-cmd)
   - [소스에서 빌드](#23-소스에서-빌드)
3. [환경 설정](#3-환경-설정)
   - [macOS 환경 변수](#31-macos-환경-변수)
   - [Windows CMD 환경 변수](#32-windows-cmd-환경-변수)
   - [HuggingFace 미러 설정](#33-huggingface-미러-설정)
4. [빠른 시작](#4-빠른-시작)
5. [컬렉션 관리](#5-컬렉션-관리)
6. [컨텍스트 관리](#6-컨텍스트-관리)
7. [임베딩 생성](#7-임베딩-생성)
8. [검색 사용법](#8-검색-사용법)
9. [문서 조회](#9-문서-조회)
10. [쿼리 문법 (QMD Syntax)](#10-쿼리-문법-qmd-syntax)
11. [MCP 서버](#11-mcp-서버)
12. [SDK 라이브러리 사용법](#12-sdk-라이브러리-사용법)
13. [출력 형식](#13-출력-형식)
14. [인덱스 유지보수](#14-인덱스-유지보수)
15. [모델 설정](#15-모델-설정)
16. [지원 파일 포맷 및 데이터 입력](#16-지원-파일-포맷-및-데이터-입력)
17. [검색 내부 동작 원리](#17-검색-내부-동작-원리)
18. [문제 해결](#18-문제-해결)

---

## 1. 시스템 요구사항

| 항목 | 최소 버전 | 비고 |
|------|-----------|------|
| **Node.js** | 22 이상 | LTS 권장 |
| **Bun** | 1.0.0 이상 | 선택적 (Node.js 대신 사용 가능) |
| **운영체제** | macOS, Linux, Windows | Windows는 x64만 지원 |
| **디스크** | ~2 GB | GGUF 모델 3개 자동 다운로드 |
| **RAM** | 4 GB 이상 | 모델 로딩에 필요 (8 GB 권장) |

### GPU 지원

| 백엔드 | 플랫폼 | 환경 변수 |
|--------|--------|-----------|
| Metal | macOS Apple Silicon (M1/M2/M3/M4) | `QMD_LLAMA_GPU=metal` |
| CUDA | NVIDIA GPU (Windows/Linux) | `QMD_LLAMA_GPU=cuda` |
| Vulkan | AMD/기타 GPU | `QMD_LLAMA_GPU=vulkan` |
| CPU | 모든 플랫폼 | `QMD_LLAMA_GPU=false` |

GPU가 없어도 CPU로 동작하지만, 임베딩/재순위화 속도가 크게 느려집니다.  
설정 없이 실행하면 GPU를 자동 감지(`auto`)합니다.

### 자동 다운로드되는 기본 GGUF 모델

| 모델 | 용도 | 크기 | 비고 |
|------|------|------|------|
| `embeddinggemma-300M-Q8_0` | 벡터 임베딩 | ~300 MB | 영어 최적화, 기본값 |
| `qwen3-reranker-0.6b-q8_0` | 재순위화 | ~640 MB | 다국어 지원 |
| `qmd-query-expansion-1.7B-q4_k_m` | 쿼리 확장 | ~1.1 GB | QMD 전용 파인튜닝 모델 |

모델은 `~/.cache/qmd/models/` (Windows: `%USERPROFILE%\.cache\qmd\models\`)에 저장됩니다.  
처음 `qmd embed` 또는 `qmd query` 실행 시 자동으로 다운로드됩니다.

---

## 2. 설치

### 2.1 macOS

#### npm으로 전역 설치

```sh
npm install -g @tobilu/qmd
```

#### Bun으로 전역 설치

```sh
bun install -g @tobilu/qmd
```

#### npx / bunx로 즉시 실행 (설치 없이)

```sh
npx @tobilu/qmd --help
bunx @tobilu/qmd --help
```

#### macOS 전용: Homebrew SQLite 설치 (필수)

macOS의 기본 SQLite는 확장 기능을 지원하지 않습니다.  
sqlite-vec 벡터 확장을 사용하려면 Homebrew SQLite가 반드시 필요합니다.

```sh
brew install sqlite
```

설치 후 셸 프로파일(`~/.zshrc` 또는 `~/.bash_profile`)에 경로를 추가합니다:

```sh
# Apple Silicon (M1/M2/M3/M4)
echo 'export PATH="/opt/homebrew/opt/sqlite/bin:$PATH"' >> ~/.zshrc
echo 'export LDFLAGS="-L/opt/homebrew/opt/sqlite/lib"' >> ~/.zshrc
echo 'export CPPFLAGS="-I/opt/homebrew/opt/sqlite/include"' >> ~/.zshrc
source ~/.zshrc

# Intel Mac
echo 'export PATH="/usr/local/opt/sqlite/bin:$PATH"' >> ~/.zshrc
echo 'export LDFLAGS="-L/usr/local/opt/sqlite/lib"' >> ~/.zshrc
echo 'export CPPFLAGS="-I/usr/local/opt/sqlite/include"' >> ~/.zshrc
source ~/.zshrc
```

설치 확인:

```sh
qmd --version
qmd status
```

---

### 2.2 Windows (CMD)

Windows는 CMD(명령 프롬프트) 환경에서 사용합니다. PowerShell도 사용 가능합니다.

#### Node.js 설치

[https://nodejs.org](https://nodejs.org) 에서 Node.js 22 LTS를 다운로드하여 설치합니다.

설치 확인:

```cmd
node --version
npm --version
```

#### Bun 설치 (선택 사항)

```cmd
powershell -c "irm bun.sh/install.ps1 | iex"
```

또는 npm으로:

```cmd
npm install -g bun
```

#### QMD 전역 설치

```cmd
npm install -g @tobilu/qmd
```

설치 확인:

```cmd
qmd --version
```

> **Windows 주의사항:** Windows에서 `bin/qmd`는 셸 스크립트(`.sh`)이므로, CMD에서는 직접 실행하지 않습니다. `npm install -g`가 올바른 래퍼를 등록합니다. 만약 `qmd` 명령이 인식되지 않으면 Node.js의 전역 bin 경로가 `PATH`에 있는지 확인하세요.

```cmd
npm config get prefix
```

출력된 경로(예: `C:\Users\사용자명\AppData\Roaming\npm`)를 시스템 `PATH`에 추가합니다.

---

### 2.3 소스에서 빌드

개발 목적이나 최신 소스를 직접 사용하려면 빌드가 필요합니다.

#### macOS에서 빌드

```sh
# 저장소 클론
git clone https://github.com/tobi/qmd
cd qmd

# 의존성 설치 (Bun 사용 권장)
bun install

# TypeScript 컴파일
npm run build

# 전역 명령으로 등록
npm link

# 확인
qmd --version
```

소스에서 바로 실행 (빌드 없이):

```sh
bun src/cli/qmd.ts <명령>
```

#### Windows CMD에서 빌드

```cmd
git clone https://github.com/tobi/qmd
cd qmd

rem 의존성 설치
npm install

rem TypeScript 컴파일
npm run build

rem 전역 명령으로 등록
npm link

rem 확인
qmd --version
```

> **중요:** Windows에서는 절대로 `bun build --compile`을 실행하지 마세요.  
> `bin/qmd`는 `dist/`의 컴파일된 JS를 실행하는 셸 스크립트입니다. 컴파일 명령이 이 래퍼를 덮어쓰면 sqlite-vec가 깨집니다.

---

## 3. 환경 설정

### 3.1 macOS 환경 변수

`~/.zshrc` 또는 `~/.bash_profile`에 추가:

```sh
# ── GPU 설정 ──────────────────────────────────────────────────────────
# metal: Apple Silicon GPU, vulkan: 범용 GPU, cuda: NVIDIA GPU
# 기본값: auto (자동 감지)
export QMD_LLAMA_GPU=metal

# ── 임베딩 모델 ────────────────────────────────────────────────────────
# 기본값: embeddinggemma-300M (영어 최적화)
# 한국어/일본어/중국어 등 다국어 지원 시 Qwen3 임베딩 권장
export QMD_EMBED_MODEL="hf:Qwen/Qwen3-Embedding-0.6B-GGUF/Qwen3-Embedding-0.6B-Q8_0.gguf"
# 로컬 파일 경로도 가능
# export QMD_EMBED_MODEL="/path/to/model.gguf"

# 임베딩 컨텍스트 크기 (기본값: 2048)
# 더 긴 문서를 임베딩하려면 늘리세요 (VRAM 증가)
export QMD_EMBED_CONTEXT_SIZE=2048

# ── 재순위 모델 ────────────────────────────────────────────────────────
# 기본값: Qwen3-Reranker-0.6B
export QMD_RERANK_MODEL="hf:ggml-org/Qwen3-Reranker-0.6B-Q8_0-GGUF/qwen3-reranker-0.6b-q8_0.gguf"

# 재순위 컨텍스트 크기 (기본값: 4096)
# 긴 문서 처리 시 늘리세요
export QMD_RERANK_CONTEXT_SIZE=4096

# ── 쿼리 확장 모델 ─────────────────────────────────────────────────────
# 기본값: qmd-query-expansion-1.7B (QMD 전용 파인튜닝)
export QMD_GENERATE_MODEL="hf:tobil/qmd-query-expansion-1.7B-gguf/qmd-query-expansion-1.7B-q4_k_m.gguf"

# 쿼리 확장 컨텍스트 크기 (기본값: 2048)
export QMD_EXPAND_CONTEXT_SIZE=2048

# ── 에디터 링크 설정 ───────────────────────────────────────────────────
# 터미널 클릭으로 에디터에서 파일 열기 (TTY 환경에서만 동작)
export QMD_EDITOR_URI="vscode://file/{path}:{line}:{col}"  # VS Code (기본값)
# export QMD_EDITOR_URI="cursor://file/{path}:{line}:{col}"  # Cursor
# export QMD_EDITOR_URI="zed://file/{path}:{line}:{col}"     # Zed

# ── 기타 ───────────────────────────────────────────────────────────────
# 캐시 디렉터리 변경 (기본값: ~/.cache)
export XDG_CACHE_HOME="$HOME/.cache"

# qmd status에서 GPU 장치 탐지 활성화 (기본값: 비활성화, 느린 장치에서는 타임아웃 가능)
export QMD_STATUS_DEVICE_PROBE=1
```

변경사항 적용:

```sh
source ~/.zshrc
```

---

### 3.2 Windows CMD 환경 변수

#### 현재 세션에서만 적용 (임시 설정)

```cmd
rem GPU 백엔드 강제 지정 (NVIDIA: cuda, AMD: vulkan)
set QMD_LLAMA_GPU=vulkan

rem 임베딩 모델 변경 (한국어 등 다국어 지원)
set QMD_EMBED_MODEL=hf:Qwen/Qwen3-Embedding-0.6B-GGUF/Qwen3-Embedding-0.6B-Q8_0.gguf

rem 에디터 링크 설정
set QMD_EDITOR_URI=vscode://file/{path}:{line}:{col}

rem 캐시 디렉터리 (기본값: %USERPROFILE%\.cache)
set XDG_CACHE_HOME=%USERPROFILE%\.cache
```

#### 영구 설정 (시스템 환경 변수)

Windows 검색에서 "시스템 환경 변수 편집" → "환경 변수" 버튼 → 사용자 변수에 추가:

| 변수명 | 값 예시 | 설명 |
|--------|---------|------|
| `QMD_LLAMA_GPU` | `vulkan` | GPU 백엔드 |
| `QMD_EMBED_MODEL` | `hf:Qwen/Qwen3-Embedding-0.6B-GGUF/...` | 임베딩 모델 |
| `QMD_RERANK_MODEL` | `hf:ggml-org/Qwen3-Reranker-0.6B-...` | 재순위 모델 |
| `QMD_GENERATE_MODEL` | `hf:tobil/qmd-query-expansion-1.7B-...` | 쿼리 확장 모델 |
| `QMD_EDITOR_URI` | `vscode://file/{path}:{line}:{col}` | 에디터 링크 |
| `XDG_CACHE_HOME` | `%USERPROFILE%\.cache` | 캐시 경로 |

또는 CMD에서 `setx` 명령으로 영구 설정:

```cmd
setx QMD_LLAMA_GPU vulkan
setx QMD_EDITOR_URI "vscode://file/{path}:{line}:{col}"
```

> `setx`는 현재 세션에는 적용되지 않습니다. 새 CMD 창을 열어서 확인하세요.

---

### 3.3 HuggingFace 미러 설정

회사 방화벽이나 지역 네트워크 문제로 huggingface.co 접속이 어려운 경우 미러를 사용할 수 있습니다.

```sh
# macOS — hf-mirror.com 사용
export HF_ENDPOINT=https://hf-mirror.com
qmd embed

# Windows CMD
set HF_ENDPOINT=https://hf-mirror.com
qmd embed
```

모델을 수동으로 다운로드하여 로컬 경로로 지정하는 방법도 있습니다:

```sh
# 수동 다운로드 후 로컬 경로 지정
export QMD_EMBED_MODEL="/home/user/.cache/qmd/models/Qwen3-Embedding-0.6B-Q8_0.gguf"
```

---

## 4. 빠른 시작

### macOS

```sh
# 1. 현재 디렉터리의 마크다운 파일로 컬렉션 생성
qmd collection add ~/notes --name notes
qmd collection add ~/Documents/meetings --name meetings

# 2. 컨텍스트 추가 (검색 품질 향상 — 선택 사항이지만 권장)
qmd context add qmd://notes "개인 노트 및 아이디어"
qmd context add qmd://meetings "회의 내용 및 회의록"

# 3. 벡터 임베딩 생성 (최초 1회, 시간이 걸림 — 모델 다운로드 포함)
qmd embed

# 4. 검색
qmd search "프로젝트 일정"           # 빠른 키워드 검색 (LLM 불필요)
qmd vsearch "배포 방법"              # 시맨틱 검색 (임베딩 필요)
qmd query "분기별 계획 프로세스"     # 하이브리드 + 재순위화 (최고 품질)
```

### Windows CMD

```cmd
rem 1. 컬렉션 생성
qmd collection add %USERPROFILE%\notes --name notes
qmd collection add %USERPROFILE%\Documents\meetings --name meetings

rem 2. 컨텍스트 추가
qmd context add qmd://notes "개인 노트 및 아이디어"
qmd context add qmd://meetings "회의 내용 및 회의록"

rem 3. 벡터 임베딩 생성
qmd embed

rem 4. 검색
qmd search "프로젝트 일정"
qmd vsearch "배포 방법"
qmd query "분기별 계획 프로세스"
```

### 검색 모드 선택 가이드

| 상황 | 권장 명령 |
|------|----------|
| 인터넷이 없거나 빠른 결과 필요 | `qmd search` |
| 의미 기반 검색, 임베딩 완료 상태 | `qmd vsearch` |
| 최고 품질 (느림, LLM 필요) | `qmd query` |
| GPU 없이 시맨틱 검색 | `qmd query --no-rerank` |

---

## 5. 컬렉션 관리

컬렉션은 QMD가 인덱싱할 디렉터리의 단위입니다. 여러 컬렉션을 등록하여 동시에 검색하거나, 특정 컬렉션만 검색할 수 있습니다.

### 컬렉션 추가

#### macOS

```sh
# 현재 디렉터리를 컬렉션으로 추가
qmd collection add . --name myproject

# 특정 경로와 이름으로 추가
qmd collection add ~/Documents/notes --name notes

# glob 패턴으로 특정 파일만 인덱싱 (기본값: **/*.md)
qmd collection add ~/work/docs --name docs --mask "**/*.md"
qmd collection add ~/src/myapp --name myapp --mask "**/*.{md,ts,js,py}"

# 여러 확장자 포함
qmd collection add ~/knowledge-base --name kb --mask "**/*.{md,txt,org}"

# 특정 파일/디렉터리 제외
qmd collection add ~/project --name project \
  --mask "**/*.md" \
  --ignore "node_modules/**" \
  --ignore "dist/**" \
  --ignore ".git/**"

# Obsidian vault 등록 예시
qmd collection add ~/ObsidianVault --name obsidian --mask "**/*.md"
```

#### Windows CMD

```cmd
qmd collection add . --name myproject
qmd collection add %USERPROFILE%\Documents\notes --name notes
qmd collection add %USERPROFILE%\work\docs --name docs --mask "**/*.md"
qmd collection add %USERPROFILE%\ObsidianVault --name obsidian --mask "**/*.md"
```

### 컬렉션 목록 확인

```sh
qmd collection list
```

출력 예시:

```
notes       ~/notes                *.md    42 docs   2026-04-24
meetings    ~/Documents/meetings   *.md    17 docs   2026-04-20
docs        ~/work/docs            *.md   128 docs   2026-04-23
```

각 컬렉션의 이름, 경로, glob 패턴, 문서 수, 마지막 인덱싱 날짜를 표시합니다.

### 컬렉션 이름 변경

```sh
qmd collection rename notes my-notes
```

### 컬렉션 삭제

```sh
# 컬렉션 등록 해제 (원본 파일은 삭제되지 않음)
qmd collection remove myproject
```

### 컬렉션 내 파일 목록

```sh
# 컬렉션 전체 파일 목록
qmd ls notes

# 특정 경로 하위 파일 목록
qmd ls notes/2025
qmd ls qmd://notes/2025

# 모든 컬렉션 목록
qmd ls
```

### YAML 설정 파일로 컬렉션 관리

반복적인 컬렉션 설정을 파일로 관리할 수 있습니다:

```yaml
# qmd.yml
collections:
  notes:
    path: ~/notes
    pattern: "**/*.md"
  work:
    path: ~/work/docs
    pattern: "**/*.{md,txt}"
    ignore:
      - "archive/**"
  code:
    path: ~/src/myapp
    pattern: "**/*.{ts,js,py}"
    chunkStrategy: auto   # 코드 파일 AST 청킹
```

```sh
# YAML 설정으로 스토어 생성 (SDK 사용 시)
const store = await createStore({ dbPath: './index.sqlite', configPath: './qmd.yml' })
```

---

## 6. 컨텍스트 관리

컨텍스트는 컬렉션이나 경로에 설명 메타데이터를 추가하여 검색 관련성과 LLM의 문서 선택 품질을 높입니다. 검색 결과와 함께 반환되어 AI 에이전트가 더 나은 선택을 할 수 있도록 합니다.

**컨텍스트가 중요한 이유:**
- 동음이의어 처리: "Java" 검색 시 "Java 커피 레시피"가 아닌 "Java 프로그래밍" 문서를 우선 반환
- AI 에이전트가 컬렉션의 내용을 사전에 파악하여 더 정확한 판단 가능
- `qmd query`의 쿼리 확장 시 컨텍스트 정보 활용

### 컨텍스트 추가

#### macOS

```sh
# 컬렉션 전체에 컨텍스트 추가 (qmd:// 가상 경로 방식)
qmd context add qmd://notes "개인 노트와 아이디어 모음"
qmd context add qmd://docs/api "REST API 참조 문서"
qmd context add qmd://meetings "팀 주간 회의 내용과 결정사항"

# 현재 디렉터리 기준으로 추가 (컬렉션 자동 감지)
cd ~/notes && qmd context add "개인 노트와 아이디어"
cd ~/notes/work && qmd context add "업무 관련 노트"

# 전역 컨텍스트 (모든 컬렉션에 공통 적용, 시스템 프롬프트 역할)
qmd context add / "회사 내부 지식 베이스. 항상 한국어로 답변하세요."

# 경로 지정 추가 — 서브디렉터리에도 적용 가능
qmd context add qmd://notes/2024 "2024년 회의록 및 노트"
qmd context add qmd://notes/2025 "2025년 회의록 및 노트"
```

#### Windows CMD

```cmd
qmd context add qmd://notes "개인 노트와 아이디어 모음"
qmd context add qmd://docs/api "REST API 참조 문서"
qmd context add / "회사 내부 지식 베이스"
```

### 컨텍스트 목록

```sh
qmd context list
```

출력 예시:

```
/                    회사 내부 지식 베이스
qmd://notes          개인 노트와 아이디어 모음
qmd://notes/2024     2024년 회의록 및 노트
qmd://docs/api       REST API 참조 문서
```

### 컨텍스트 누락 확인

```sh
qmd context check
```

컨텍스트가 없는 컬렉션이나 경로를 표시합니다.

### 컨텍스트 삭제

```sh
qmd context rm qmd://notes/old-folder
qmd context rm /    # 전역 컨텍스트 삭제
```

---

## 7. 임베딩 생성

벡터 검색(`vsearch`, `query`)을 사용하려면 반드시 임베딩을 생성해야 합니다.  
임베딩은 각 문서 청크를 고차원 벡터로 변환하여 SQLite에 저장합니다.

```sh
# 인덱싱된 문서의 임베딩 생성 (최초 1회 또는 신규 문서 추가 후)
qmd embed

# 강제 전체 재생성 (임베딩 모델 변경 후 필수)
qmd embed -f

# AST 기반 청킹 활성화 (코드 파일에서 함수/클래스 단위로 청킹)
# TypeScript, JavaScript, Python, Go, Rust 지원
qmd embed --chunk-strategy auto

# 특정 컬렉션만 임베딩
qmd embed -c notes
```

### 청킹 전략

| 전략 | 옵션 | 설명 |
|------|------|------|
| `regex` | (기본값) | 정규식 기반 청킹. 마크다운 헤딩을 청크 경계로 사용 |
| `auto` | `--chunk-strategy auto` | 코드 파일은 AST(tree-sitter) 기반 함수/클래스 단위 청킹 |

- 청크 크기: 900 토큰, 15% 오버랩
- 마크다운/미지원 파일은 항상 `regex` 전략 사용
- 임베딩 모델을 변경(`QMD_EMBED_MODEL`)한 경우 반드시 `qmd embed -f`로 전체 재생성하세요 (벡터 차원이 달라 호환되지 않음)

---

## 8. 검색 사용법

QMD는 세 가지 검색 모드를 제공합니다:

| 명령 | 방식 | 속도 | 특징 |
|------|------|------|------|
| `qmd search` | BM25 전문 검색 | 매우 빠름 | LLM 불필요, 키워드 정확 일치 |
| `qmd vsearch` | 벡터 시맨틱 검색 | 보통 | 의미 기반, 임베딩 필요 |
| `qmd query` | 하이브리드 (BM25 + 벡터 + 재순위화) | 느림 | 최고 품질, 권장 |

### 기본 검색

#### macOS

```sh
# 키워드 검색 (빠름, LLM 없이도 동작)
qmd search "인증 흐름"

# 시맨틱 검색 (임베딩 모델 사용)
qmd vsearch "로그인하는 방법"

# 하이브리드 검색 (권장 — 쿼리 확장 + BM25 + 벡터 + 재순위화)
qmd query "사용자 인증"
```

#### Windows CMD

```cmd
qmd search "인증 흐름"
qmd vsearch "로그인하는 방법"
qmd query "사용자 인증"
```

### 검색 옵션

```sh
# 결과 수 지정 (기본값: 5)
qmd query -n 10 "API 디자인 패턴"
qmd search -n 20 "에러 처리"

# 특정 컬렉션만 검색
qmd query -c notes "회의 내용"
qmd search --collection docs "API"

# 최소 점수 필터링 (0.0 ~ 1.0)
qmd query --min-score 0.3 "에러 처리"

# 모든 결과 반환 (결과 수 제한 없음)
qmd query --all --min-score 0.4 "에러 처리"

# 문서 전체 내용 표시 (스니펫 대신 전문)
qmd search --full "인증"

# 라인 번호 포함
qmd search --line-numbers "함수 정의"

# 재순위화 건너뛰기 (속도 우선, GPU 없을 때 유용)
qmd query --no-rerank "검색어"

# 점수 산출 과정 상세 표시 (디버깅용)
qmd query --explain "검색어"

# JSON 출력으로 점수 산출 과정 포함
qmd query --json --explain "검색어"
```

### 다중 라인 쿼리

#### macOS (bash/zsh의 `$'...'` 문법 사용)

```sh
# 키워드 + 시맨틱 조합 쿼리
qmd query $'lex: 인증 토큰\nvec: 사용자 인증은 어떻게 동작하나요'

# 키워드 + 시맨틱 + 가상답변 조합
qmd query $'lex: rate limiter\nvec: API 속도 제한이 어떻게 동작하나요\nhyde: API는 슬라이딩 윈도우 알고리즘을 사용합니다. 클라이언트가 분당 100개 초과 시 429 반환...'

# intent(검색 의도)와 함께
qmd query $'intent: 웹 성능 및 Core Web Vitals\nlex: performance\nvec: 성능을 개선하는 방법'
```

#### macOS (`--intent` 플래그 사용)

```sh
# 단일 키워드에 검색 의도 추가
qmd query --intent "웹 성능 및 지연 시간" "performance"

# intent + 복합 쿼리
qmd query --intent "Spring Boot 마이크로서비스" $'lex: service discovery\nvec: 서비스 간 통신 방법'
```

#### Windows CMD (멀티라인 쿼리)

CMD에서는 `$'...'` 문법을 지원하지 않으므로 `--intent` 플래그나 단순 쿼리를 사용합니다:

```cmd
rem --intent 플래그로 intent 지정 (권장)
qmd query --intent "웹 성능 및 지연 시간" "performance"

rem 단순 키워드 검색
qmd search "인증 토큰"

rem 특정 컬렉션 검색
qmd query -c docs "API 디자인"
```

> Windows CMD에서 복잡한 다중 라인 QMD 쿼리가 필요한 경우, MCP 서버(HTTP 모드)를 통해 JSON 형식으로 전송하거나 SDK를 사용하세요.

---

## 9. 문서 조회

### 문서 ID (docid)

각 문서는 고유한 짧은 ID(docid)를 가집니다 — 내용 해시의 앞 6자리입니다.  
검색 결과에서 `#abc123` 형태로 표시되며, `get` 및 `multi-get` 명령에 사용할 수 있습니다.

```sh
# 검색으로 docid 확인
qmd search "인증" --json
# 출력: [{"docid": "#abc123", "score": 0.85, "file": "docs/auth.md", ...}]

# docid로 문서 조회
qmd get "#abc123"
qmd get abc123    # 앞의 # 생략 가능
```

### 단일 문서 조회

#### macOS

```sh
# 경로로 조회
qmd get "docs/readme.md"

# 가상 경로(qmd://)로 조회
qmd get "qmd://notes/meeting.md"

# 특정 라인부터 조회 (경로:라인번호)
qmd get "notes/meeting.md:50"

# 최대 라인 수 제한
qmd get "notes/meeting.md" -l 100

# 시작 라인 + 라인 수 지정
qmd get "notes/meeting.md" --from 50 -l 100
```

#### Windows CMD

```cmd
qmd get "docs\readme.md"
qmd get "#abc123"
qmd get "notes\meeting.md" -l 100
qmd get "notes\meeting.md" --from 50 -l 100
```

### 다중 문서 조회

#### macOS

```sh
# glob 패턴으로 여러 문서 조회
qmd multi-get "journals/2025-05*.md"

# 쉼표로 구분된 목록 (경로, docid 혼용 가능)
qmd multi-get "doc1.md, doc2.md, #abc123"

# 파일 크기 제한 (기본값: 10 KB, 초과 파일은 건너뜀)
qmd multi-get "docs/*.md" --max-bytes 20480

# 문서당 최대 라인 수
qmd multi-get "docs/*.md" -l 200

# JSON 출력
qmd multi-get "docs/*.md" --json

# 마크다운 출력 (LLM에 컨텍스트로 제공 시)
qmd multi-get "docs/*.md" --md
```

#### Windows CMD

```cmd
qmd multi-get "journals\2025-05*.md"
qmd multi-get "doc1.md, doc2.md, #abc123"
qmd multi-get "docs\*.md" --max-bytes 20480 --json
```

---

## 10. 쿼리 문법 (QMD Syntax)

QMD 쿼리는 타입이 지정된 서브쿼리로 구성된 구조화된 문서입니다.  
각 줄은 `타입: 내용` 형식으로 작성합니다.

### 쿼리 유형

| 타입 | 백엔드 | 설명 |
|------|--------|------|
| `lex` | BM25 | 키워드 검색 (정확한 일치, 빠름) |
| `vec` | 벡터 | 시맨틱 유사도 검색 (자연어 질문) |
| `hyde` | 벡터 | 가상 답변 임베딩 검색 (예상 답변 형태로 작성) |
| `expand` | 자동 확장 | LLM이 lex/vec/hyde 변형을 자동 생성 |
| `intent` | 컨텍스트 | 검색 의도 명시 (직접 검색되지 않음) |

### 단일 라인 쿼리 (기본 확장 쿼리)

단순 텍스트를 입력하면 LLM이 자동으로 lex/vec/hyde 변형을 생성합니다:

```
# 이 두 형식은 동일합니다
how does authentication work
expand: how does authentication work
```

### Lex 쿼리 특수 문법

| 문법 | 의미 | 예시 |
|------|------|------|
| `word` | 접두사 일치 | `perf` → "performance", "perfmon" 포함 |
| `"phrase"` | 정확한 구문 일치 | `"rate limiter"` |
| `-word` | 단어 제외 | `-sports` |
| `-"phrase"` | 구문 제외 | `-"test data"` |

```sh
# 예시
qmd search "CAP theorem consistency"
qmd search '"machine learning" -"deep learning"'
qmd search "auth -oauth -saml"
```

### 다중 타입 조합 쿼리

첫 번째 쿼리가 RRF 융합에서 2배 가중치를 받습니다.

```
lex: rate limiter algorithm
vec: API에서 속도 제한이 어떻게 동작하나요
hyde: API는 토큰 버킷 알고리즘을 구현합니다. 클라이언트가 분당 100개 요청을 초과하면 429 응답을 반환합니다.
```

```
lex: kubernetes pod restart
vec: 파드가 CrashLoopBackOff 상태일 때 디버깅 방법
hyde: kubectl describe pod <name>으로 이벤트 확인 후 로그를 분석합니다...
```

### intent 사용

모호한 쿼리를 명확하게 합니다. 검색 파이프라인에 문맥 정보를 제공하며 직접 검색되지는 않습니다.

```
intent: 웹 페이지 로딩 시간과 Core Web Vitals 최적화
lex: performance
vec: 성능을 개선하는 방법
```

### 규칙

- `expand:` 와 `lex:`/`vec:`/`hyde:` 는 동일 쿼리 문서에서 혼용 불가
- `intent:` 는 쿼리 문서당 최대 1개, 단독 사용 불가 (다른 타입과 함께 사용)
- 빈 줄은 무시됨, 앞뒤 공백 제거됨

---

## 11. MCP 서버

QMD는 AI 에이전트(Claude Desktop, Cursor, VS Code 등)와 연동하기 위한 MCP(Model Context Protocol) 서버를 제공합니다.

### 노출되는 MCP 도구

| 도구 | 설명 |
|------|------|
| `query` | lex/vec/hyde 서브쿼리 + RRF + 재순위화 검색 |
| `get` | 경로 또는 docid로 문서 조회 |
| `multi_get` | glob 패턴이나 목록으로 여러 문서 조회 |
| `status` | 인덱스 상태 및 컬렉션 정보 |

### stdio 모드 (클라이언트가 서브프로세스로 실행)

이 모드에서는 AI 에이전트가 qmd 프로세스를 직접 실행합니다. 매번 모델을 로딩하므로 첫 응답이 느릴 수 있습니다.

#### Claude Desktop 설정

설정 파일 위치:
- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "qmd": {
      "command": "qmd",
      "args": ["mcp"]
    }
  }
}
```

#### Cursor 설정

```json
{
  "mcpServers": {
    "qmd": {
      "command": "qmd",
      "args": ["mcp"],
      "env": {
        "QMD_EMBED_MODEL": "hf:Qwen/Qwen3-Embedding-0.6B-GGUF/Qwen3-Embedding-0.6B-Q8_0.gguf"
      }
    }
  }
}
```

### HTTP 모드 (공유 장기 실행 서버)

모델을 VRAM에 유지하여 반복 로딩 없이 빠른 응답이 가능합니다.  
여러 AI 에이전트나 동시 요청에서 특히 유리합니다.

#### macOS

```sh
# 포그라운드 실행 (Ctrl+C로 중지)
qmd mcp --http                    # localhost:8181
qmd mcp --http --port 8080        # 포트 지정

# 백그라운드 데몬으로 실행
qmd mcp --http --daemon           # PID를 ~/.cache/qmd/mcp.pid에 저장
qmd mcp stop                      # 데몬 중지
qmd status                        # 상태 확인 ("MCP: running (PID ...)" 표시)
```

#### Windows CMD

```cmd
rem 포그라운드 실행
qmd mcp --http
qmd mcp --http --port 8080

rem 데몬 모드 (백그라운드)
qmd mcp --http --daemon
qmd mcp stop
qmd status
```

MCP 클라이언트에서 `http://localhost:8181/mcp`로 연결합니다.

엔드포인트:
- `POST /mcp` — MCP Streamable HTTP (JSON, 상태 비저장)
- `GET /health` — 헬스 체크 (업타임 포함)

#### HTTP 모드 MCP 클라이언트 설정 (Claude Desktop)

```json
{
  "mcpServers": {
    "qmd": {
      "type": "http",
      "url": "http://localhost:8181/mcp"
    }
  }
}
```

### JSON 형식으로 쿼리 (MCP/HTTP API)

단순 쿼리:

```json
{
  "q": "lex: CAP theorem\nvec: consistency vs availability",
  "collections": ["docs"],
  "limit": 10
}
```

구조화된 형식:

```json
{
  "searches": [
    { "type": "lex", "query": "CAP theorem" },
    { "type": "vec", "query": "consistency vs availability" }
  ],
  "intent": "분산 시스템 설계",
  "limit": 5,
  "minScore": 0.3
}
```

---

## 12. SDK 라이브러리 사용법

QMD를 Node.js / Bun 애플리케이션에서 라이브러리로 사용할 수 있습니다.

### 설치

```sh
npm install @tobilu/qmd
# 또는
bun add @tobilu/qmd
```

### 기본 사용법

```typescript
import { createStore } from '@tobilu/qmd'

const store = await createStore({
  dbPath: './my-index.sqlite',
  config: {
    collections: {
      docs: { path: '/path/to/docs', pattern: '**/*.md' },
    },
  },
})

const results = await store.search({ query: "인증 흐름" })
console.log(results.map(r => `${r.title} (${Math.round(r.score * 100)}%)`))

await store.close()
```

### 스토어 생성 모드

```typescript
// 1. 인라인 config — 새 스토어 생성
const store = await createStore({
  dbPath: './index.sqlite',
  config: {
    collections: {
      docs: { path: '/path/to/docs', pattern: '**/*.md' },
      notes: { path: '/path/to/notes' },
    },
  },
})

// 2. YAML config 파일 사용
const store2 = await createStore({
  dbPath: './index.sqlite',
  configPath: './qmd.yml',
})

// 3. DB만 지정 (이전에 설정된 스토어 재오픈)
const store3 = await createStore({ dbPath: './index.sqlite' })
```

### 검색 API

```typescript
// 단순 쿼리 (자동 확장 + BM25 + 벡터 + 재순위화)
const results = await store.search({ query: "인증 흐름" })

// 전체 옵션 사용
const results2 = await store.search({
  query: "rate limiting",
  intent: "API 속도 제한 및 남용 방지",
  collection: "docs",
  limit: 5,
  minScore: 0.3,
  rerank: true,       // 재순위화 활성화 (기본값: true)
  explain: true,      // 점수 산출 과정 포함
})

// 사전 구성된 쿼리 (자동 확장 건너뜀)
const results3 = await store.search({
  queries: [
    { type: 'lex', query: '"connection pool" timeout -redis' },
    { type: 'vec', query: '데이터베이스 연결이 과부하에서 왜 타임아웃되나요' },
    { type: 'hyde', query: '데이터베이스 커넥션 풀이 소진되면 새 요청은 대기하다 타임아웃됩니다...' },
  ],
  collections: ["docs", "notes"],
})

// 재순위화 건너뛰기 (빠른 결과)
const fast = await store.search({ query: "auth", rerank: false })
```

### 직접 백엔드 접근

```typescript
// BM25 키워드 검색 (빠름, LLM 없음)
const lexResults = await store.searchLex("auth middleware", { limit: 10 })

// 벡터 유사도 검색 (임베딩 모델 사용, 재순위화 없음)
const vecResults = await store.searchVector("사용자가 로그인하는 방법", { limit: 10 })

// 수동 쿼리 확장
const expanded = await store.expandQuery("auth flow", { intent: "user login" })
const results4 = await store.search({ queries: expanded })
```

### 문서 조회 API

```typescript
// 경로 또는 docid로 조회
const doc = await store.get("docs/readme.md")
const byId = await store.get("#abc123")

if (!("error" in doc)) {
  console.log(doc.title, doc.displayPath, doc.context)
}

// 라인 범위 지정 조회
const body = await store.getDocumentBody("docs/readme.md", {
  fromLine: 50,
  maxLines: 100,
})

// 일괄 조회 (glob 패턴)
const { docs, errors } = await store.multiGet("docs/**/*.md", {
  maxBytes: 20480,   // 20 KB 초과 파일 건너뜀
})
```

### 컬렉션 API

```typescript
// 컬렉션 추가
await store.addCollection("myapp", {
  path: "/src/myapp",
  pattern: "**/*.ts",
  ignore: ["node_modules/**", "*.test.ts"],
})

// 컬렉션 목록 조회
const collections = await store.listCollections()

// 컬렉션 삭제 / 이름 변경
await store.removeCollection("myapp")
await store.renameCollection("old-name", "new-name")
```

### 인덱싱 API

```typescript
// 파일시스템 스캔으로 컬렉션 재인덱싱
const result = await store.update({
  collections: ["docs"],           // 생략 시 전체 컬렉션
  onProgress: ({ collection, file, current, total }) => {
    process.stdout.write(`\r[${collection}] ${current}/${total} ${file}`)
  },
})

// 벡터 임베딩 생성
const embedResult = await store.embed({
  force: false,                    // true이면 전체 재생성
  chunkStrategy: "auto",           // "regex" 또는 "auto" (코드 파일 AST 청킹)
  collections: ["docs"],           // 생략 시 전체 컬렉션
  onProgress: ({ current, total, collection }) => {
    process.stdout.write(`\r임베딩 중 ${current}/${total} [${collection}]`)
  },
})
```

### 스토어 종료

```typescript
// 반드시 호출 — LLM 모델과 DB 연결 해제. 미호출 시 NAPI 크래시 가능
await store.close()
```

---

## 13. 출력 형식

### 기본 출력 (터미널)

```
docs/guide.md:42 #a1b2c3
Title: Software Craftsmanship
Context: 업무 문서
Score: 93%

This section covers the craftsmanship of building
quality software with attention to detail.
```

- **Score**: 색상 코딩 (녹색 >70%, 노란색 >40%, 회색 그 이하)
- **터미널 링크**: TTY 환경에서 파일 경로 클릭 시 `QMD_EDITOR_URI`로 에디터 열기
- **--explain 포함 시**: BM25 점수, 벡터 점수, RRF 점수, 재순위 점수 분해 표시

### 출력 형식 옵션

```sh
# 파일 목록만 출력 (에이전트 워크플로우용, 최소 출력)
qmd search "API" --files
# 출력: docid,score,filepath,context

# JSON 출력
qmd query "분기 보고서" --json

# 점수 산출 과정 포함 JSON
qmd query "분기 보고서" --json --explain

# CSV 출력
qmd search "API" --csv

# 마크다운 출력 (LLM 컨텍스트로 전달 시 유용)
qmd search --md --full "에러 처리"

# XML 출력
qmd search "API" --xml
```

### JSON 출력 구조

```json
[
  {
    "docid": "#a1b2c3",
    "score": 0.93,
    "file": "docs/guide.md",
    "title": "Software Craftsmanship",
    "context": "업무 문서",
    "snippet": "This section covers...",
    "explain": {
      "bm25": 0.72,
      "vector": 0.88,
      "rrf": 0.91,
      "rerank": 0.93
    }
  }
]
```

### NO_COLOR 지원

```sh
# macOS
export NO_COLOR=1
qmd query "검색어"

# Windows CMD
set NO_COLOR=1
qmd query "검색어"
```

---

## 14. 인덱스 유지보수

### 인덱스 상태 확인

```sh
qmd status
```

표시 내용:
- 컬렉션 목록, 문서 수, 마지막 인덱싱 시간
- MCP 서버 실행 상태 (`running (PID ...)`)
- 사용 가능한 tree-sitter 문법 (AST 청킹 지원 언어)
- GPU 장치 정보 (`QMD_STATUS_DEVICE_PROBE=1` 필요)
- 임베딩 통계 (임베딩된 청크 수)

### 재인덱싱

```sh
# 모든 컬렉션 재인덱싱 (파일 변경 감지)
qmd update

# git pull 후 재인덱싱 (컬렉션이 git 저장소인 경우)
qmd update --pull

# 특정 컬렉션만 재인덱싱
qmd update -c notes
```

### 캐시 정리

```sh
# 캐시 및 고아 데이터 정리 (삭제된 파일의 임베딩 등)
qmd cleanup
```

### 데이터 저장 위치

| 항목 | macOS / Linux | Windows |
|------|---------------|---------|
| 인덱스 DB | `~/.cache/qmd/index.sqlite` | `%USERPROFILE%\.cache\qmd\index.sqlite` |
| GGUF 모델 | `~/.cache/qmd/models/` | `%USERPROFILE%\.cache\qmd\models\` |
| MCP PID | `~/.cache/qmd/mcp.pid` | `%USERPROFILE%\.cache\qmd\mcp.pid` |

> SQLite 데이터베이스를 직접 수정하지 마세요. 항상 qmd 명령을 통해 관리하세요.

---

## 15. 모델 설정

QMD는 세 가지 역할의 GGUF 모델을 사용하며, 환경 변수 또는 YAML 설정으로 교체 가능합니다.

### 기본 모델 요약

| 역할 | 기본 모델 | 크기 | 특징 |
|------|-----------|------|------|
| 임베딩 | `embeddinggemma-300M-Q8_0` | ~300 MB | 영어 최적화, 빠름 |
| 재순위 | `Qwen3-Reranker-0.6B-Q8_0` | ~640 MB | 다국어 지원 |
| 쿼리 확장 | `qmd-query-expansion-1.7B-q4_k_m` | ~1.1 GB | QMD 전용 파인튜닝 |

### 환경 변수로 모델 변경

```sh
# macOS ~/.zshrc

# 임베딩 모델
export QMD_EMBED_MODEL="hf:<user>/<repo>/<filename>.gguf"

# 재순위 모델
export QMD_RERANK_MODEL="hf:<user>/<repo>/<filename>.gguf"

# 쿼리 확장 모델
export QMD_GENERATE_MODEL="hf:<user>/<repo>/<filename>.gguf"

# 모델 변경 후 전체 재임베딩 (임베딩 모델 변경 시만 필수)
qmd embed -f
```

```cmd
rem Windows CMD
set QMD_EMBED_MODEL=hf:<user>/<repo>/<filename>.gguf
qmd embed -f
```

### 임베딩 모델 선택 가이드

| 모델 | 크기 | 다국어 | 권장 용도 |
|------|------|--------|----------|
| `embeddinggemma-300M-Q8_0` (기본) | ~300 MB | 영어 최적화 | 영어 문서, 빠른 임베딩 필요 시 |
| `Qwen3-Embedding-0.6B-Q8_0` | ~640 MB | 우수 | 한국어/일본어/중국어 문서, 다국어 |
| `Qwen3-Embedding-4B-Q4_K_M` | ~2.5 GB | 우수 | 최고 품질, 충분한 VRAM 보유 시 |

#### 한국어/다국어 문서 검색 (Qwen3 임베딩 권장)

기본 `embeddinggemma-300M`은 영어 최적화 모델입니다.  
**한국어, 일본어, 중국어 등 다국어 문서**에는 Qwen3-Embedding 사용을 강력히 권장합니다.

```sh
# macOS — 0.6B (가벼움, 다국어 지원)
export QMD_EMBED_MODEL="hf:Qwen/Qwen3-Embedding-0.6B-GGUF/Qwen3-Embedding-0.6B-Q8_0.gguf"
qmd embed -f    # 임베딩 차원이 달라지므로 전체 재생성 필수

# macOS — 4B (더 높은 품질, 약 2.5 GB)
export QMD_EMBED_MODEL="hf:Qwen/Qwen3-Embedding-4B-GGUF/Qwen3-Embedding-4B-Q4_K_M.gguf"
qmd embed -f
```

```cmd
rem Windows CMD
set QMD_EMBED_MODEL=hf:Qwen/Qwen3-Embedding-0.6B-GGUF/Qwen3-Embedding-0.6B-Q8_0.gguf
qmd embed -f
```

#### Qwen3-Embedding 특이사항

Qwen3-Embedding은 쿼리와 문서의 임베딩 포맷이 다릅니다:
- **쿼리**: `Instruct: Retrieve relevant documents for the given query\nQuery: <쿼리>` 형식
- **문서**: 원시 텍스트 (특별한 접두사 없음)

QMD는 이를 자동으로 감지하여 처리합니다. 별도 설정이 필요 없습니다.

### 재순위 모델 선택 가이드

| 모델 | 크기 | 특징 |
|------|------|------|
| `Qwen3-Reranker-0.6B-Q8_0` (기본) | ~640 MB | 다국어 지원, 빠름 |
| `Qwen3-Reranker-1.5B-Q8_0` | ~1.6 GB | 더 높은 정확도 |

```sh
# 1.5B 재순위 모델 사용 (더 높은 품질)
export QMD_RERANK_MODEL="hf:Qwen/Qwen3-Reranker-1.5B-GGUF/Qwen3-Reranker-1.5B-Q8_0.gguf"
```

### 쿼리 확장 모델 선택 가이드

| 모델 | 크기 | 특징 |
|------|------|------|
| `qmd-query-expansion-1.7B-q4_k_m` (기본) | ~1.1 GB | QMD 전용 파인튜닝, 빠름 |
| `LFM2-1.2B-Q4_K_M` | ~0.8 GB | LiquidAI 하이브리드 아키텍처, 엣지 최적화 |
| `Qwen3-0.6B-Q8_0` | ~640 MB | 일반 목적 언어 모델 |

```sh
# LiquidAI LFM2 사용 (가벼운 엣지 환경)
export QMD_GENERATE_MODEL="hf:LiquidAI/LFM2-1.2B-GGUF/LFM2-1.2B-Q4_K_M.gguf"

# Qwen3-0.6B 기본 사용
export QMD_GENERATE_MODEL="hf:ggml-org/Qwen3-0.6B-GGUF/Qwen3-0.6B-Q8_0.gguf"
```

### 최신 모델에 대하여

QMD는 node-llama-cpp를 통해 HuggingFace의 **모든 GGUF 형식 모델**을 지원합니다.  
단, 역할별로 모델 타입이 맞아야 합니다:

- **임베딩 모델**: embedding 컨텍스트를 지원하는 GGUF (예: Qwen3-Embedding, nomic-embed, E5 등)
- **재순위 모델**: ranking 컨텍스트를 지원하는 GGUF (예: Qwen3-Reranker, bge-reranker 등)
- **쿼리 확장 모델**: 인스트럭션 튜닝된 텍스트 생성 GGUF (Qwen3 호환 권장)

> **Gemma 4 / Gemma 임베딩 모델:** `ggml-org`에서 공개된 Gemma 계열 임베딩 GGUF가 있다면 아래 형식으로 사용 가능합니다:
> ```sh
> export QMD_EMBED_MODEL="hf:ggml-org/<gemma-embedding-model-repo>/<file>.gguf"
> ```
> 공식 지원 여부는 [ggml-org HuggingFace](https://huggingface.co/ggml-org)에서 확인하세요.

### YAML 설정 파일로 컬렉션별 모델 지정

`index.yml`의 `models:` 섹션을 사용하면 컬렉션마다 다른 모델을 사용할 수 있습니다:

```yaml
collections:
  # 영어 문서: 기본 embeddinggemma 사용
  docs-en:
    path: ~/docs/english
    pattern: "**/*.md"

  # 한국어 문서: Qwen3-Embedding으로 더 높은 품질
  docs-ko:
    path: ~/docs/korean
    pattern: "**/*.md"
    models:
      embed: "hf:Qwen/Qwen3-Embedding-0.6B-GGUF/Qwen3-Embedding-0.6B-Q8_0.gguf"

  # 코드베이스: AST 청킹 + 특화 임베딩
  code:
    path: ~/src/myapp
    pattern: "**/*.{ts,py}"
    chunkStrategy: auto
```

모델 결정 우선순위: **YAML `models:` 섹션** > **환경 변수** > **내장 기본값**

### 컨텍스트 크기 튜닝

```sh
# 임베딩 컨텍스트 (기본값: 2048 토큰)
# 긴 문서 청크를 정확히 임베딩하려면 늘리세요 (VRAM 사용량 증가)
export QMD_EMBED_CONTEXT_SIZE=4096

# 재순위 컨텍스트 (기본값: 4096 토큰)
# 매우 긴 문서 처리 시 늘리세요
export QMD_RERANK_CONTEXT_SIZE=8192

# 쿼리 확장 컨텍스트 (기본값: 2048 토큰)
# 쿼리 확장 모델의 응답 생성 공간
export QMD_EXPAND_CONTEXT_SIZE=2048
```

### GPU 설정

```sh
# macOS (Apple Silicon M1/M2/M3/M4)
export QMD_LLAMA_GPU=metal

# NVIDIA GPU (Windows/Linux)
export QMD_LLAMA_GPU=cuda

# AMD GPU (Vulkan, Windows/Linux)
export QMD_LLAMA_GPU=vulkan

# CPU 강제 (GPU 비활성화)
export QMD_LLAMA_GPU=false

# 자동 감지 (기본값, 설정 불필요)
# export QMD_LLAMA_GPU=auto
```

```cmd
rem Windows CMD - NVIDIA
set QMD_LLAMA_GPU=cuda

rem Windows CMD - AMD/기타
set QMD_LLAMA_GPU=vulkan

rem Windows CMD - CPU만 사용
set QMD_LLAMA_GPU=false
```

---

## 16. 지원 파일 포맷 및 데이터 입력

### 데이터 입력 방법

데이터 입력은 **"파일을 디렉터리에 놓고 컬렉션으로 등록"** 하는 것이 전부입니다.  
QMD 자체에는 에디터나 업로드 UI가 없습니다. 파일을 직접 작성하거나 복사한 뒤 아래 명령을 실행하면 변경사항이 반영됩니다.

```sh
qmd collection add ~/my-notes --name notes
qmd update    # 파일 스캔 및 인덱싱
qmd embed     # 벡터 임베딩 생성
```

### 지원 파일 포맷

#### 완전 지원 (제목 파싱 포함)

| 포맷 | 확장자 | 제목 추출 방식 |
|------|--------|---------------|
| Markdown | `.md` | `# 제목` 또는 `## 제목` 헤딩 |
| Org-mode | `.org` | `#+TITLE:` 또는 `* 헤딩` |

#### 코드 파일 지원 (AST 청킹)

| 언어 | 확장자 | AST 청킹 대상 |
|------|--------|--------------|
| TypeScript | `.ts` `.tsx` `.mts` `.cts` | 함수, 클래스, 임포트 블록 |
| JavaScript | `.js` `.jsx` `.mjs` `.cjs` | 함수, 클래스, 임포트 블록 |
| Python | `.py` | 함수, 클래스 |
| Go | `.go` | 함수, 메서드, 타입 |
| Rust | `.rs` | 함수, impl 블록, 구조체 |

제목은 파일명으로 처리되고, `--chunk-strategy auto` 옵션 사용 시 함수/클래스 단위로 청킹됩니다.

#### 기타 텍스트 파일 (부분 지원)

UTF-8 텍스트라면 glob 패턴에 추가하면 인덱싱은 되지만 제목 파싱 없이 파일명이 제목으로 사용됩니다.

```sh
# .txt, .org 파일도 함께 인덱싱
qmd collection add ~/notes --name notes --mask "**/*.{md,txt,org}"

# YAML, TOML 설정 파일도 가능
qmd collection add ~/config --name config --mask "**/*.{yaml,toml,json}"
```

> 기본 glob 패턴은 `**/*.md`입니다. 다른 확장자를 추가하려면 컬렉션 추가 시 `--mask` 옵션으로 명시해야 합니다.

#### 미지원 포맷

| 포맷 | 이유 | 우회 방법 |
|------|------|---------|
| `.pdf` | 바이너리 | `pdftotext` 또는 `marker`로 변환 |
| `.docx` `.pptx` `.xlsx` | 바이너리 | LibreOffice export로 텍스트 변환 |
| `.png` `.jpg` `.webp` 등 이미지 | 바이너리, OCR 없음 | Tesseract OCR로 변환 |
| `.epub` | 바이너리 | Calibre로 txt/html 변환 |

> 내부적으로 `readFileSync(filepath, "utf-8")`로 읽으므로 **UTF-8 인코딩 텍스트 파일만 처리**됩니다.

### 이미지 기반 문서 (PDF, 스캔본 등) 처리 방법

QMD에는 OCR이나 PDF 파서가 내장되어 있지 않습니다. 사전 변환 후 인덱싱해야 합니다.

#### 방법 1: PDF → 텍스트/마크다운 변환

**macOS:**
```sh
# pdftotext (Homebrew poppler) — 간단한 텍스트 추출
brew install poppler
pdftotext document.pdf document.md

# marker — AI 기반 고품질 변환 (표, 코드블록, 수식 보존)
pip install marker-pdf
marker_single document.pdf output_dir/
```

**Windows CMD:**
```cmd
rem poppler for Windows 설치 후 (https://github.com/oschwartz10612/poppler-windows)
pdftotext.exe document.pdf document.md
```

#### 방법 2: 스캔 이미지 → OCR → 텍스트

**macOS:**
```sh
brew install tesseract tesseract-lang
tesseract scan.png output -l kor        # 한국어
tesseract scan.png output -l kor+eng    # 한국어 + 영어
```

**Windows CMD:**
```cmd
rem Tesseract 설치 후 (https://github.com/UB-Mannheim/tesseract/wiki)
tesseract scan.png output -l kor
```

#### 방법 3: Obsidian / Notion / Bear → 마크다운 내보내기

Obsidian, Notion, Bear, Logseq 등은 모두 마크다운 내보내기를 지원합니다.  
내보낸 디렉터리를 그대로 컬렉션으로 등록하면 바로 사용 가능합니다.

```sh
# 예: Obsidian vault 등록
qmd collection add ~/ObsidianVault --name obsidian
qmd update
qmd embed
```

---

## 17. 검색 내부 동작 원리

QMD의 검색 파이프라인을 이해하면 더 효과적으로 사용할 수 있습니다.

### qmd query 파이프라인

```
입력 쿼리
    │
    ▼
[1] 쿼리 확장 (LLM)
    qmd-query-expansion 모델이 lex/vec/hyde 변형 생성
    예) "인증 흐름" → lex: "auth token", vec: "사용자 인증 동작 방식", hyde: "JWT 토큰을 검증하는..."
    │
    ├─────────────────────────────────────┐
    ▼                                     ▼
[2a] BM25 검색 (SQLite FTS5)          [2b] 벡터 검색 (sqlite-vec)
     lex 쿼리로 키워드 매칭               vec/hyde 쿼리로 코사인 유사도 검색
     (빠름, 정확한 단어 일치)             (의미 기반, 임베딩 필요)
    │                                     │
    └──────────────┬──────────────────────┘
                   ▼
           [3] RRF 융합
               Reciprocal Rank Fusion으로 두 결과 통합
               첫 번째 쿼리에 2배 가중치 적용
               │
               ▼
           [4] Qwen3 재순위화
               쿼리-문서 관련성을 LLM이 직접 평가
               최종 점수 결정
               │
               ▼
           최종 결과
```

### BM25 (qmd search)

SQLite FTS5 기반의 전통적인 키워드 검색입니다.  
- LLM 없이 동작하므로 즉각적 응답
- 정확한 키워드 일치에 강함
- 동의어나 의미적 유사성은 처리 못함

### 벡터 검색 (qmd vsearch)

sqlite-vec를 이용한 코사인 유사도 검색입니다.  
- 의미가 비슷한 문서를 찾는 데 강함 ("로그인 방법" → "인증 프로세스" 매칭)
- 임베딩 생성(`qmd embed`)이 선행되어야 함
- 임베딩 모델 품질이 결과에 직접 영향

### RRF (Reciprocal Rank Fusion)

BM25와 벡터 검색의 순위를 통합하는 알고리즘입니다.  
- 두 검색에서 상위에 랭크된 문서를 더 높이 점수 부여
- 어느 한 방식에만 의존하는 것보다 강건한 결과 생성

### 재순위화 (Reranking)

Qwen3-Reranker가 쿼리-문서 쌍을 직접 평가합니다.  
- "이 문서가 이 쿼리에 얼마나 관련 있는가"를 LLM이 직접 판단
- 가장 느리지만 가장 정확한 단계
- `--no-rerank`로 건너뛸 수 있음 (속도 우선 시)

---

## 18. 문제 해결

### macOS — `dlopen` 또는 SQLite 확장 오류

**원인**: 시스템 SQLite가 확장을 지원하지 않음  
**해결**: Homebrew SQLite 설치 후 PATH 설정

```sh
brew install sqlite
export PATH="/opt/homebrew/opt/sqlite/bin:$PATH"  # Apple Silicon
# export PATH="/usr/local/opt/sqlite/bin:$PATH"   # Intel Mac
source ~/.zshrc
```

### Windows — `qmd` 명령을 찾을 수 없음

**해결**: npm 전역 bin 경로를 PATH에 추가

```cmd
npm config get prefix
rem 출력된 경로(예: C:\Users\사용자\AppData\Roaming\npm)를 시스템 PATH에 추가
```

시스템 환경 변수 편집 → PATH에 위 경로 추가 → CMD 재시작

### Windows — ABI 불일치 오류 (`better-sqlite3`, `sqlite-vec`)

**원인**: 패키지 설치 시 사용한 런타임(Node.js 또는 Bun)과 실행 시 런타임이 다름  
**해결**: 설치와 실행에 동일한 런타임 사용

```cmd
rem npm으로 설치했으면 node로 실행됨 (자동)
npm install -g @tobilu/qmd

rem bun으로 설치했으면 bun으로 실행됨 (자동)
bun install -g @tobilu/qmd
```

### 모델 다운로드 실패 — HTML 파일 오류

**원인**: 방화벽, 프록시, 또는 네트워크 문제로 HuggingFace 연결 실패  
**해결**: HuggingFace 미러 사용 또는 수동 다운로드

```sh
# 미러 사용 (macOS)
HF_ENDPOINT=https://hf-mirror.com qmd embed

# Windows CMD
set HF_ENDPOINT=https://hf-mirror.com
qmd embed

# 수동 다운로드 후 로컬 경로 지정
export QMD_EMBED_MODEL="/path/to/downloaded/model.gguf"
```

### 임베딩이 무한 루프에 빠짐

**해결**:

```sh
# Ctrl+C로 중단 후 재실행
qmd embed
```

### 임베딩 모델 변경 후 차원 불일치 오류

**해결**: 강제 전체 재임베딩

```sh
qmd embed -f
```

모델마다 벡터 차원이 다릅니다(예: embeddinggemma=1536, Qwen3-Embedding=1024). 임베딩 모델 변경 후 반드시 `-f` 옵션으로 전체 재생성해야 합니다.

### GPU 드라이버 문제로 `qmd status` 오류

**해결**: GPU 장치 탐지 비활성화 (기본값)

```sh
# 환경 변수가 없으면 기본적으로 GPU 탐지 건너뜀 (안전)
# 필요 시 활성화:
export QMD_STATUS_DEVICE_PROBE=1  # macOS
# set QMD_STATUS_DEVICE_PROBE=1    # Windows CMD
```

GPU 초기화 자체가 실패하는 경우:

```sh
export QMD_LLAMA_GPU=false   # GPU 완전 비활성화, CPU로만 실행
```

### 모델 다운로드 위치 확인

```sh
# macOS / Linux
ls ~/.cache/qmd/models/

# Windows CMD
dir %USERPROFILE%\.cache\qmd\models\
```

### 인덱스 초기화 (처음부터 다시)

> **주의**: 아래 명령은 되돌릴 수 없습니다. 컬렉션 설정, 컨텍스트, 임베딩이 모두 삭제됩니다.

```sh
# macOS / Linux
rm ~/.cache/qmd/index.sqlite

# Windows CMD
del %USERPROFILE%\.cache\qmd\index.sqlite
```

이후 컬렉션 추가, `qmd update`, `qmd embed` 순서로 재설정합니다.

### 성능 최적화 팁

| 상황 | 해결책 |
|------|--------|
| 임베딩이 너무 느림 | `QMD_LLAMA_GPU` 설정 확인, GPU 활용 여부 확인 |
| `qmd query`가 느림 | `--no-rerank` 사용, 또는 HTTP 모드 MCP 서버 사용 |
| 메모리 부족 | 더 작은 양자화 모델 사용 (Q4_K_M 대신 Q2_K 등) |
| 검색 품질 낮음 | 한국어는 Qwen3-Embedding으로 변경, 컨텍스트 추가 |
| 결과가 너무 적음 | `--min-score` 낮추기 또는 `--all` 사용 |

---

## 참고 자료

- [QMD GitHub](https://github.com/tobi/qmd)
- [쿼리 문법 상세 문서](SYNTAX.md)
- [CHANGELOG](../CHANGELOG.md)
- npm 패키지: [@tobilu/qmd](https://www.npmjs.com/package/@tobilu/qmd)
- GGUF 모델 검색: [huggingface.co/ggml-org](https://huggingface.co/ggml-org)
- HuggingFace 미러: [hf-mirror.com](https://hf-mirror.com)
