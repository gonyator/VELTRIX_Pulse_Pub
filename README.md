# VELTRIX Pulse v2.8

> Pulse the Attack Surface - Mass URL Health Check & Signal Detector

**VELTRIX Pulse**는 수백~수천 개의 URL에 대한 가용성(Alive) 점검, 리다이렉트 추적, 인증/관리자/개발환경 노출 신호를 빠르게 식별하는 GUI 기반 대량 점검 도구입니다.

[![Python Version](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-2.8-orange.svg)](https://github.com/<ORG>/VELTRIX_Pulse)

---

[![image](https://raw.githubusercontent.com/gonyator/VELTRIX_Pulse_Pub/main/example.png)](https://raw.githubusercontent.com/gonyator/VELTRIX_Pulse_Pub/main/example.png)


## 📑 목차

- [핵심 특징](#-핵심-특징)
- [아키텍처](#-아키텍처)
- [설치 방법](#-설치-방법)
- [사용 가이드](#-사용-가이드)
- [스캔 옵션 상세](#-스캔-옵션-상세)
- [플러그인 시스템](#-플러그인-시스템)
- [리포트 형식](#-리포트-형식)
- [고급 설정](#-고급-설정)
- [성능 튜닝](#-성능-튜닝)
- [트러블슈팅](#-트러블슈팅)
- [🧭 코드 기준 문서 정합성 점검 (2026-04-02)](#-코드-기준-문서-정합성-점검-2026-04-02)

---

## 🎯 핵심 특징

### 1. 대량 처리 최적화
- **다중 워커 기반**: ThreadPoolExecutor로 10~100개 동시 점검
- **스마트 큐 관리**: 우선순위 기반 URL 처리
- **메모리 효율**: 스트리밍 방식으로 대용량 URL 목록 처리

### 2. 신호 기반 우선순위
- 🔐 **Auth Signal**: 로그인 필요 페이지 자동 식별
- 🛠 **Admin Probe**: 관리자 경로 노출 탐지
- 🧪 **Dev/Stg Crawl**: 개발/테스트 환경 힌트 수집
- 🌐 **Subdomain Enum**: DNS 기반 서브도메인 후보 점검

### 3. 리다이렉트 인텔리전스
- 30x 체인 전체 추적 (최대 10 홉)
- 최종 도달 URL 및 상태 코드 기록
- 리다이렉트 패턴 분석 (HTTP→HTTPS, www 추가 등)

### 4. 시각화 및 리포트
- HTML 대시보드 (통계 카드, 플래그 집계, 결과 테이블)
- CSV 엑셀 친화적 출력
- 스크린샷 자동 캡처 (Playwright 통합)

---

## 🏗 아키텍처

```
┌─────────────────────────────────────────────────────┐
│              main.py + gui.py (Tkinter)             │
│                                                     │
│  [URL 목록 입력] → [옵션 선택] → [Start Scan]      │
└─────────────────┬───────────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────────┐
│                  scanner.py                         │
│                                                     │
│  ┌───────────────────────────────────────────┐     │
│  │  ThreadPoolExecutor (Workers: 10~100)     │     │
│  │                                           │     │
│  │  URL Queue → Worker → HTTP Request        │     │
│  │                ↓                          │     │
│  │         Check Status (200, 404, 500...)   │     │
│  │                ↓                          │     │
│  │         Track Redirects (30x chain)       │     │
│  │                ↓                          │     │
│  │         Extract Response Data             │     │
│  └───────────────────────────────────────────┘     │
└─────────┬───────────┬───────────┬───────────┬───────┘
          │           │           │           │
   ┌──────▼─────┐ ┌──▼──────┐ ┌──▼─────┐ ┌───▼──────┐
   │  redirect  │ │ crawler │ │subdomain│ │screenshot│
   │    .py     │ │  .py    │ │  .py    │ │   .py    │
   │            │ │         │ │         │ │          │
   │ -  체인추적 │ │ -  Dev   │ │ -  DNS   │ │ -  Playw- │
   │ -  루프감지 │ │   링크  │ │   조회  │ │   right  │
   └────────────┘ └─────────┘ └─────────┘ └──────────┘
          │           │           │           │
          └───────────┴───────────┴───────────┘
                       │
               ┌───────▼────────┐
               │  flags_rules   │
               │      .py       │
               │                │
               │ -  Auth Signal  │
               │ -  Admin Signal │
               │ -  Dev/Stg Sig  │
               │ -  Error Sig    │
               └────────┬───────┘
                        │
                ┌───────▼────────┐
                │   report.py    │
                │                │
                │ -  HTML (Stats) │
                │ -  CSV          │
                └────────────────┘
```

---

## 📦 설치 방법

### 기본 설치

```bash
# 저장소 클론
git clone https://github.com/<ORG>/VELTRIX_Pulse.git
cd VELTRIX_Pulse

# 가상환경 생성 (권장)
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 의존성 설치
pip install -r requirements.txt
```

### 의존성 패키지 (requirements.txt)

```
requests>=2.28.0
urllib3>=1.26.0
beautifulsoup4>=4.11.0
dnspython>=2.2.0
jinja2>=3.0.0
```

### 선택 설치 (스크린샷 기능)

```bash
pip install playwright
playwright install chromium
```

---

## 🚀 사용 가이드

### Windows 간편 실행

```cmd
run.bat
```

`run.bat` 내용:
```batch
@echo off
echo ========================================
echo VELTRIX Pulse v2.8 Launcher
echo Project Maintainers
echo ========================================
echo.

python main.py

if errorlevel 1 (
    echo.
    echo [ERROR] Failed to launch VELTRIX Pulse
    echo Please check Python installation
    pause
)
```

### 수동 실행

```bash
python main.py
```

### GUI 사용 절차

1. **URL 목록 준비** (`urls.txt`)
```
https://example.com
https://api.example.com
https://staging.example.com
https://admin.example.com
http://dev.example.com:8080
```

2. **GUI 실행 후 설정**
   - **Browse**: `urls.txt` 파일 선택
   - **Follow Redirects**: ✅ (30x 체인 추적)
   - **Max Redirects**: 10
   - **Workers**: 20 (동시 처리 수)
   - **Depth**: 2 (크롤링 깊이, 옵션)

3. **옵션 선택**
   - ⬜ **Enable Subdomain Enum**: 서브도메인 점검
   - ⬜ **Enable Screenshot**: 스크린샷 캡처
   - ✅ **Enable Dev-Stg Crawl**: 개발 환경 링크 수집
   - ✅ **Enable Admin Probe**: 관리자 경로 점검

4. **Start Scan** 클릭

5. **결과 확인**
   - `report_20260214_095600.html` (대시보드)
   - `report_20260214_095600.csv` (엑셀 가능)
   - `screenshots/` (스크린샷, 옵션 사용 시)

---

## 🔍 스캔 옵션 상세

### 1. Follow Redirects (리다이렉트 추적)

**동작:**
```
https://example.com
    ↓ 301 Moved Permanently
https://www.example.com
    ↓ 302 Found
https://www.example.com/en/
    ↓ 200 OK (최종 도달)
```

**출력:**
```csv
Original URL,Final URL,Status,Redirect Chain
https://example.com,https://www.example.com/en/,200,"301 → 302 → 200"
```

**설정:**
- **Max Redirects**: 추적할 최대 홉 수 (권장: 10)
- 무한 루프 감지: 동일 URL 재방문 시 중단

### 2. Workers (동시 처리 수)

**권장 값:**
- **소규모 (< 100 URL)**: 10~20 워커
- **중규모 (100~1000 URL)**: 20~50 워커
- **대규모 (1000+ URL)**: 50~100 워커

**주의:**
- 너무 높은 값은 타겟 서버 부하 유발
- 네트워크 대역폭 고려
- 메모리 사용량 증가

### 3. Enable Subdomain Enum (서브도메인 열거)

**탐지 방법:**
- **DNS 조회**: A, AAAA, CNAME 레코드
- **와일드카드 점검**: `*.example.com` 패턴 확인
- **브루트포스**: wordlists/subdomains.txt 사용

**워드리스트 예시 (`wordlists/subdomains.txt`):**
```
www
api
admin
dev
staging
test
portal
mail
ftp
vpn
```

**출력:**
```
[Subdomain] www.example.com (Resolved: 203.0.113.10)
[Subdomain] api.example.com (Resolved: 203.0.113.20)
[Subdomain] admin.example.com (Auth Required ⚠️)
```

### 4. Enable Screenshot (스크린샷)

**요구사항:**
```bash
pip install playwright
playwright install chromium
```

**동작:**
- Headless Chrome으로 각 URL 렌더링
- 1280x720 해상도 캡처
- `screenshots/` 디렉터리에 저장
- 파일명: `example_com_20260214_095600.png`

**설정 (screenshot.py):**
```python
SCREENSHOT_CONFIG = {
    "viewport": {"width": 1280, "height": 720},
    "timeout": 30000,  # 30초
    "wait_until": "networkidle",  # 네트워크 유휴 상태 대기
    "format": "png",
    "quality": 80  # JPEG 품질 (1-100)
}
```

**스크린샷/다운로드 분류 규칙 (운영 기준):**
- `screenshot_ok`: HTML 렌더링이 가능하고 스크린샷 파일이 생성된 경우
- `no_screenshot`: 렌더링/캡처가 실패했고 다운로드 이벤트도 없었던 경우
- `download_on_visit`: 페이지 접속 중 브라우저 다운로드 이벤트가 발생한 경우(스크린샷 생성 여부와 무관하게 다운로드 메타를 우선 기록)

**경계 케이스 처리 기준:**
- 리다이렉트 후 즉시 파일 다운로드: `download_on_visit`
- 인증 실패(로그인/403) + 다운로드 없음: `no_screenshot` 또는 일반 페이지 캡처 성공 시 `screenshot_ok`
- 빈 응답/브라우저 오류/타임아웃 + 다운로드 없음: `no_screenshot`
- 빈 응답처럼 보여도 다운로드 이벤트가 관측되면: `download_on_visit`

**다운로드 메타데이터 필드:**
- `id`, `filename`, `mime_type`, `size`, `path`, `created_at`
- API/결과 직렬화 호환 필드: `downloadAvailable`(기본 `false`), `downloadId`/`downloadUrl`(기본 `null`)

### 5. Enable Dev-Stg Crawl (개발 환경 크롤링)

**목적:**
- 프로덕션 페이지에서 개발/테스트 환경 링크 힌트 수집
- 실수로 노출된 내부 환경 발견

**탐지 패턴 (crawler.py):**
```python
DEV_PATTERNS = [
    r'https?://[^/]*dev[^/]*\.',
    r'https?://[^/]*staging[^/]*\.',
    r'https?://[^/]*test[^/]*\.',
    r'https?://localhost[:/]',
    r'https?://127\.0\.0\.1[:/]',
    r'https?://192\.168\.',
    r'\.local/',
    r'/debug',
    r'/test/',
]
```

**출력:**
```
[Dev-Stg] Found: http://dev.example.com/api
  Source: https://example.com/contact.html (line 142)
  Risk: Medium (Internal environment exposed)
```

### 6. Enable Admin Probe (관리자 경로 점검)

**워드리스트 (`admin_probe_paths.txt`):**
```
/admin
/administrator
/admin.php
/admin/login
/admin/dashboard
/wp-admin
/phpmyadmin
/administrator/index.php
/admin/cp
/controlpanel
/adminpanel
/admin1
/admin2
/_admin
/wp-login.php
```

**동작:**
```
https://example.com + /admin
    ↓ 403 Forbidden → Flag: Admin path exists but protected
    ↓ 200 OK + Login form → Flag: Admin login exposed
    ↓ 302 → /admin/dashboard → Flag: Redirect chain (follow)
    ↓ 404 Not Found → (skip)
```

**출력:**
```
[Admin Probe] /admin → 403 Forbidden
  Flag: Protected admin path detected
  Risk: Low (Access denied)

[Admin Probe] /phpmyadmin → 200 OK
  Flag: Database admin exposed
  Risk: High (No authentication required ⚠️)
```

---

## 🔌 플러그인 시스템

## 🧭 코드 기준 문서 정합성 점검 (2026-04-02)

`VELTRIX_Pulse_v2` 전체 로직(엔트리포인트/스캐너/플러그인/리포트)을 기준으로 README 정합성을 점검한 결과, 아래 항목은 **문서 업데이트 권장** 상태입니다.

### 1) 실행 경로/명령어 정합성
- 현재 실제 실행 엔트리포인트는 `VELTRIX_Pulse_v2/main.py`입니다.
- 따라서 설치/실행 예시는 `cd VELTRIX_Pulse_v2` 후 `python main.py` 기준으로 맞추는 것이 정확합니다.

### 2) 필수 의존성 표기 보완
- 코드에서 `jinja2`를 직접 사용(`report.py`)하고 시작 시 의존성 검사(`main.py`)도 수행하므로, 필수 의존성 목록에 `jinja2`를 포함해야 합니다.

### 3) 플러그인 예시 인터페이스 최신화
- 문서의 `run_plugin(url, response_data)` 방식 예시는 현재 코드 구조와 다릅니다.
- 실제 플러그인은 `plugins/*.py` 내 `PLUGIN` 객체(클래스)로 등록되며, `on_start / process_target / on_finish` 훅 기반으로 동작합니다.

### 4) 플래그/리포트 설명 범위 확장
- 현재 리포트 집계는 `AUTH, MFA, ADMIN, DEV/STG, API, BACKUP, CONFIG, DEBUG, UPLOAD, DATABASE, INFO` 및 다운로드 감지(`FILE_DL`)까지 확장되어 있습니다.
- README의 플래그 설명이 일부 축약되어 있어 최신 플래그 셋 기준으로 업데이트가 필요합니다.

### 5) 스캔 동작 설명(보안/신뢰도) 보강
- 코어 스캔은 SSL 검증을 무시(`verify=False`)하는 안전모드 성격의 동작을 사용합니다.
- 실행 후 `coverage / reliability / plugin_report` 등의 런 메타가 생성되므로, 리포트 해석 가이드에 해당 필드를 추가하면 운영 시 오해를 줄일 수 있습니다.

### 6) 옵션 기본값 문서화 권장
- GUI 기본값: `workers=30`, `max_redirects=5`, `crawl_depth=1`, `subdomain=off`, `screenshot/on`, `crawl/on`, `admin_probe/on`
- README의 일부 예시값(예: max redirects 10, workers 20)은 “예시”임을 명시하거나 “기본값”과 분리 표기하는 것이 좋습니다.

---

### 플러그인 구조

```
plugins/
├── __init__.py
├── custom_checker.py
└── api_fuzzer.py
```

### 커스텀 플러그인 작성

**plugins/custom_checker.py:**
```python
"""
VELTRIX Pulse Plugin: Custom Security Checker
Author: Your Name
"""

def plugin_info():
    """플러그인 메타데이터"""
    return {
        "name": "Custom Security Checker",
        "version": "1.0",
        "description": "Custom vulnerability checks",
        "author": "Your Name"
    }

def run_plugin(url: str, response_data: dict) -> dict:
    """
    플러그인 메인 로직

    Args:
        url: 타겟 URL
        response_data: {
            "status": 200,
            "headers": {...},
            "body": "...",
            "final_url": "..."
        }

    Returns:
        {
            "findings": [
                {
                    "type": "Custom Vuln",
                    "severity": "High",
                    "description": "...",
                    "evidence": "..."
                }
            ]
        }
    """
    findings = []

    # 커스텀 로직
    if "X-Debug-Mode: true" in response_data.get("headers", {}):
        findings.append({
            "type": "Debug Mode Enabled",
            "severity": "Medium",
            "description": "X-Debug-Mode header detected",
            "evidence": "X-Debug-Mode: true"
        })

    return {"findings": findings}
```

### 플러그인 로딩 (scanner.py)

```python
# scanner.py 내부

import importlib
import os

def load_plugins():
    """plugins/ 디렉터리에서 플러그인 로드"""
    plugins = []
    plugin_dir = os.path.join(os.path.dirname(__file__), 'plugins')

    for filename in os.listdir(plugin_dir):
        if filename.endswith('.py') and filename != '__init__.py':
            module_name = filename[:-3]
            try:
                module = importlib.import_module(f'plugins.{module_name}')
                if hasattr(module, 'run_plugin'):
                    info = module.plugin_info() if hasattr(module, 'plugin_info') else {}
                    plugins.append({
                        "module": module,
                        "info": info
                    })
                    print(f"[Plugin] Loaded: {info.get('name', module_name)}")
            except Exception as e:
                print(f"[Plugin] Failed to load {module_name}: {e}")

    return plugins

# 스캔 시 플러그인 실행
plugins = load_plugins()
for plugin in plugins:
    try:
        result = plugin["module"].run_plugin(url, response_data)
        if result.get("findings"):
            report_data["plugin_findings"].extend(result["findings"])
    except Exception as e:
        print(f"[Plugin] Error in {plugin['info'].get('name')}: {e}")
```

---

## 📊 리포트 형식

### HTML 리포트 (report.py)

**구조:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>VELTRIX Pulse Report</title>
    <style>
        /* Bootstrap + Custom CSS */
        .stats-card { background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); }
        .flag-high { color: #dc3545; font-weight: bold; }
        .flag-medium { color: #ffc107; }
        .flag-low { color: #28a745; }
    </style>
</head>
<body>
    <div class="container">
        <!-- 1. Header -->
        <h1>VELTRIX Pulse Report</h1>
        <p>Generated: 2026-02-14 09:56:00</p>
        <p>Total URLs: 1,234 | Alive: 987 | Dead: 247</p>

        <!-- 2. Statistics Cards -->
        <div class="row">
            <div class="col-md-3">
                <div class="stats-card">
                    <h3>987</h3>
                    <p>Alive URLs</p>
                </div>
            </div>
            <div class="col-md-3">
                <div class="stats-card">
                    <h3>42</h3>
                    <p>Auth Required</p>
                </div>
            </div>
            <div class="col-md-3">
                <div class="stats-card">
                    <h3>15</h3>
                    <p>Admin Paths</p>
                </div>
            </div>
            <div class="col-md-3">
                <div class="stats-card">
                    <h3>8</h3>
                    <p>Dev/Stg Links</p>
                </div>
            </div>
        </div>

        <!-- 3. Flags Summary -->
        <h2>Flags Summary</h2>
        <table class="table">
            <tr>
                <td class="flag-high">Auth Signal</td>
                <td>42 URLs</td>
            </tr>
            <tr>
                <td class="flag-high">Admin Exposed</td>
                <td>15 URLs</td>
            </tr>
            <tr>
                <td class="flag-medium">Dev/Stg Links</td>
                <td>8 URLs</td>
            </tr>
        </table>

        <!-- 4. Detailed Results -->
        <h2>Detailed Results</h2>
        <table class="table table-striped">
            <thead>
                <tr>
                    <th>Original URL</th>
                    <th>Final URL</th>
                    <th>Status</th>
                    <th>Flags</th>
                    <th>Screenshot</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>https://example.com</td>
                    <td>https://www.example.com/</td>
                    <td><span class="badge badge-success">200 OK</span></td>
                    <td>-</td>
                    <td><a href="screenshots/example_com.png">View</a></td>
                </tr>
                <tr>
                    <td>https://example.com/admin</td>
                    <td>https://example.com/admin/login</td>
                    <td><span class="badge badge-success">200 OK</span></td>
                    <td><span class="flag-high">Admin, Auth</span></td>
                    <td><a href="screenshots/example_com_admin.png">View</a></td>
                </tr>
            </tbody>
        </table>

        <!-- 5. Plugin Results -->
        <h2>Plugin Results</h2>
        <ul>
            <li><strong>Custom Security Checker</strong>: 3 findings</li>
            <li><strong>API Fuzzer</strong>: 12 findings</li>
        </ul>
    </div>
</body>
</html>
```

### CSV 리포트

```csv
Original URL,Final URL,Status,Redirect Chain,Flags,Title,Content Length,Response Time
https://example.com,https://www.example.com/,200,"301 → 200",-,Example Domain,1234,0.234
https://example.com/admin,https://example.com/admin/login,200,"302 → 200","Admin, Auth",Admin Login,5678,0.456
https://api.example.com,https://api.example.com/,403,-,Auth Required,403 Forbidden,234,0.123
https://staging.example.com,https://staging.example.com/,200,-,Dev/Stg,Staging Environment,9012,0.789
```

---

## ⚙️ 고급 설정

### metadata.py

```python
"""
VELTRIX Pulse Metadata
"""

VERSION = "2.8"
AUTHOR = "Gonyator"
DESCRIPTION = "Mass URL health check and signal detector"

# 기본 설정
DEFAULT_CONFIG = {
    "workers": 20,
    "timeout": 10,
    "max_redirects": 10,
    "depth": 2,
    "user_agent": f"VELTRIX Pulse/{VERSION}",
    "verify_ssl": True,
}

# 플래그 정의
FLAGS = {
    "AUTH": {"color": "red", "priority": 1},
    "ADMIN": {"color": "red", "priority": 1},
    "DEV_STG": {"color": "orange", "priority": 2},
    "ERROR": {"color": "yellow", "priority": 3},
}
```

### flags_rules.py (플래그 룰 엔진)

```python
"""
VELTRIX Pulse Flags Rules Engine
"""

import re

def apply_flags(url: str, response_data: dict) -> list:
    """
    URL과 응답 데이터에 플래그 적용

    Returns:
        ["AUTH", "ADMIN"] 형태의 플래그 리스트
    """
    flags = []
    status = response_data.get("status")
    headers = response_data.get("headers", {})
    body = response_data.get("body", "")

    # 1. Auth Signal
    if status == 401:
        flags.append("AUTH")
    elif status == 403:
        flags.append("AUTH")
    elif "WWW-Authenticate" in headers:
        flags.append("AUTH")
    elif any(pattern in body.lower() for pattern in [
        "login", "sign in", "authentication required", "please log in"
    ]):
        flags.append("AUTH")

    # 2. Admin Signal
    admin_patterns = [
        r'/admin', r'/administrator', r'/wp-admin', r'/phpmyadmin',
        r'/controlpanel', r'/adminpanel', r'/manage'
    ]
    if any(re.search(pattern, url, re.I) for pattern in admin_patterns):
        flags.append("ADMIN")

    # 3. Dev/Stg Signal
    dev_patterns = [
        r'://dev\.', r'://staging\.', r'://test\.', r'\.dev\.', r'\.staging\.',
        r'localhost', r'127\.0\.0\.1', r'192\.168\.'
    ]
    if any(re.search(pattern, url, re.I) for pattern in dev_patterns):
        flags.append("DEV_STG")

    # 4. Error Signal
    if status >= 500:
        flags.append("ERROR")
    elif "error" in body.lower() or "exception" in body.lower():
        flags.append("ERROR")

    return list(set(flags))  # 중복 제거
```

---

## 🚄 성능 튜닝

### 1. 워커 수 최적화

**CPU 코어 기반:**
```python
import os
optimal_workers = os.cpu_count() * 2  # 하이퍼스레딩 고려
```

**네트워크 기반:**
```python
# 네트워크 대역폭이 병목인 경우
# 테스트로 최적값 찾기
for workers in [10, 20, 50, 100]:
    duration = benchmark_scan(urls, workers)
    print(f"Workers: {workers} → Duration: {duration}s")
```

### 2. 타임아웃 조정

타임아웃은 중앙 설정 파일이 아닌, 각 모듈별 파일에서 개별 수정합니다.

| 모듈 | 파일 | 변수 | 기본값 | 설명 |
|------|------|------|--------|------|
| Flag 스캔 | `plugins/flags.py` | `TIMEOUT` | 15 | 플래그 판별 HTTP 요청 |
| Subdomain Flag | `plugins/subdomain.py` | `FLAGS_TIMEOUT` | 12 | 서브도메인 플래그 스캔 |
| Admin Probe | `plugins/base.py` | `admin_probe_timeout` | 3.0 | 관리자 경로 점검 |
| Crawler | `crawler.py` | `CRAWL_TIMEOUT` | 15 | Dev/Stg 크롤링 |
| Redirect 연결 | `redirect.py` | `CONNECT_TIMEOUT` | 2 | TCP 연결 타임아웃 |
| Redirect 읽기 | `redirect.py` | `READ_TIMEOUT` | 3 | 응답 읽기 타임아웃 |
| Screenshot | `screenshot.py` | `SCREENSHOT_TIMEOUT` | 30000 (ms) | 스크린샷 캡처 |
| DNS 조회 | `subdomain.py` | `resolver.timeout` | 0.5 | DNS 쿼리 타임아웃 |

**튜닝 가이드:**
```python
# 빠른 스캔 (대용량 처리) — 각 파일에서 값을 낮춤
# 예: plugins/flags.py → TIMEOUT = 5
#     crawler.py       → CRAWL_TIMEOUT = 5

# 안정적 스캔 (정확도 우선) — 기본값 유지 또는 소폭 상향
# 예: plugins/flags.py → TIMEOUT = 15 (기본값)

# 응답 느린 타겟 — 각 파일에서 값을 높임
# 예: plugins/flags.py → TIMEOUT = 30
#     redirect.py      → READ_TIMEOUT = 10
```

### 3. 배치 처리

**대용량 URL 목록 (10,000+ URLs):**
```python
def batch_scan(urls, batch_size=1000):
    """URL 목록을 배치로 나누어 처리"""
    for i in range(0, len(urls), batch_size):
        batch = urls[i:i+batch_size]
        print(f"Processing batch {i//batch_size + 1}...")
        scan_urls(batch)
        # 배치 간 쿨다운
        time.sleep(5)
```

### 4. 메모리 관리

```python
# 스트리밍 방식 URL 읽기
def read_urls_streaming(filepath):
    """메모리 효율적 URL 읽기"""
    with open(filepath, 'r') as f:
        for line in f:
            url = line.strip()
            if url and url.startswith('http'):
                yield url

# 사용
for url in read_urls_streaming('large_urls.txt'):
    process_url(url)
```

---

## 🔧 트러블슈팅

### 1. 워커 과부하 (Too many open files)

**문제:**
```
OSError: [Errno 24] Too many open files
```

**해결:**
```bash
# Linux/macOS: ulimit 증가
ulimit -n 4096

# 영구 설정 (/etc/security/limits.conf)
* soft nofile 4096
* hard nofile 8192
```

**코드 레벨:**
```python
# utils.py
import resource
resource.setrlimit(resource.RLIMIT_NOFILE, (4096, 8192))
```

### 2. Playwright 미설치

**문제:**
```
playwright._impl._api_types.Error: Executable doesn't exist
```

**해결:**
```bash
pip install playwright
playwright install chromium

# 특정 버전
playwright install chromium --with-deps
```

### 3. SSL 인증서 오류

**문제:**
```
requests.exceptions.SSLError: [SSL: CERTIFICATE_VERIFY_FAILED]
```

**해결:**
```python
# scanner.py
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# requests 호출 시
response = requests.get(url, verify=False, timeout=10)
```

### 4. GUI 미실행 (Tkinter)

**문제:**
```
ImportError: No module named '_tkinter'
```

**해결:**
```bash
# Ubuntu/Debian
sudo apt install python3-tk

# macOS
brew install python-tk

# Windows: Python 재설치 시 "tcl/tk" 옵션 체크
```

### 5. 메모리 부족

**문제:**
```
MemoryError
```

**해결:**
1. 워커 수 감소: 100 → 50 → 20
2. 배치 크기 감소: 1000 → 500 → 100
3. 스크린샷 비활성화 (메모리 집약적)
4. 시스템 스왑 증가

### 6. DNS 조회 실패

**문제:**
```
dns.resolver.NoNameservers: All nameservers failed
```

**해결:**
```python
# subdomain.py
import dns.resolver

# 커스텀 DNS 서버 지정
resolver = dns.resolver.Resolver()
resolver.nameservers = ['8.8.8.8', '1.1.1.1']  # Google, Cloudflare DNS
```

---

## 📈 사용 사례

### Case 1: 자산 가시성 확보 (Asset Discovery)

**시나리오:**
- 기업의 모든 웹 자산 현황 파악
- 어떤 서비스가 살아있고, 어떤 것이 죽었는지

**실행:**
```bash
# 1. 전체 자산 목록 준비 (DNS 기록, 과거 스캔 결과 등)
# assets.txt: 5,000개 URL

# 2. VELTRIX Pulse 실행
python main.py
# - Workers: 50
# - Follow Redirects: Yes
# - Subdomain Enum: Yes

# 3. 결과 분석
# - Alive: 3,245
# - Dead: 1,755
# - Redirect 패턴: HTTP→HTTPS (80%), www 추가 (15%)
```

**리포트 활용:**
- 죽은 서비스 폐기 (비용 절감)
- 리다이렉트 정리 (SEO 개선)
- 서브도메인 인벤토리 업데이트

### Case 2: 버그바운티 Recon

**시나리오:**
- `example.com` 대상 버그바운티
- 숨겨진 관리자 경로, 개발 환경 찾기

**실행:**
```bash
# 1. 서브도메인 수집
python main.py
# - Input: example.com (단일 도메인)
# - Subdomain Enum: Yes (wordlist: 10,000개)
# - Output: 42개 서브도메인 발견

# 2. 전체 URL 재점검
# subdomains.txt + 관리자 경로 조합
# admin.example.com/login
# api.example.com/admin
# staging.example.com/admin

python main.py
# - Admin Probe: Yes
# - Dev-Stg Crawl: Yes
# - Screenshot: Yes
```

**발견 사항:**
```
[CRITICAL] https://staging.example.com/admin
  Status: 200 OK
  Flags: Admin, Dev/Stg, No Auth
  Title: Admin Dashboard
  Screenshot: ✅

[HIGH] https://api.example.com/swagger-ui.html
  Status: 200 OK
  Flags: API Spec Exposed
  Endpoints: 127개
```

### Case 3: 레거시 시스템 정리

**시나리오:**
- 10년간 축적된 1,000개 내부 서비스
- 사용 중인 것과 폐기 대상 구분

**실행:**
```bash
# 1. 전체 스캔
python main.py
# - Input: internal_services.txt (1,000 URLs)
# - Workers: 30
# - Timeout: 5 (빠른 스캔)

# 2. 결과 필터링 (CSV)
import pandas as pd
df = pd.read_csv('report.csv')

# Dead 서비스
dead = df[df['Status'].isin([0, 502, 503, 504])]
print(f"Dead services: {len(dead)}")

# Auth 없는 서비스
no_auth = df[~df['Flags'].str.contains('AUTH', na=False)]
print(f"No auth: {len(no_auth)}")  # 보안 위험
```

---

## 📝 라이선스

MIT License

---

## 🤝 기여 가이드

플러그인 기여 환영!

1. `plugins/` 디렉터리에 `.py` 파일 추가
2. `plugin_info()`, `run_plugin()` 함수 구현
3. Pull Request

---

## 📞 Project Maintainers

- **운영 주체**: Gonyator (https://www.youtube.com/@gonyator8998/videos)
- **Official Repository**: https://github.com/gonyator/VELTRIX_Pulse
- **Issues**: https://github.com/gonyator/VELTRIX_Pulse/issues

---

## ⚠️ 법적 고지

VELTRIX Pulse는 **합법적이고 승인된** 자산 점검 목적으로만 사용해야 합니다. 권한 없는 시스템에 대한 스캔은 법률 위반입니다.
