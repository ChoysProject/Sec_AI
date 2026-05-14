
### 🔗 3단계: 화면 접속 및 모델 연동

모든 엔진이 리눅스(WSL) 안에서 돌아가고 있습니다. 이제 화면을 띄워 연결할 차례입니다.

1.  **브라우저 접속**: Windows에서 크롬이나 엣지를 열고 주소창에 `http://localhost`를 입력합니다.
2.  **초기 설정**: Dify 관리자 계정을 생성하고 로그인합니다.
3.  **모델 연동**: 우측 상단 프로필 클릭 ➡ **Settings** ➡ **Model Provider** ➡ **Ollama**를 선택합니다.
    *   **LLM 연동**:
        *   Model Name: `qwen2.5:0.5b`
        *   Base URL: `http://host.docker.internal:11434` (Docker Desktop 환경에서 WSL과 통신하는 만능 주소입니다)
        *   Model Type: `Chat`
    *   **임베딩 연동** (Ollama 항목에서 'Add Model'을 한 번 더 누릅니다):
        *   Model Name: `bge-m3`
        *   Base URL: `http://host.docker.internal:11434`
        *   Model Type: `Text Embedding`

이제 상단의 **'Knowledge'** 메뉴로 이동해 텍스트 파일을 업로드(임베딩 테스트) 해보시거나, **'Studio'** 메뉴에서 챗봇을 만들어좋습니다! 개인 Windows 노트북과 WSL(Windows Subsystem for Linux)을 활용하여, **인터넷이 완전히 차단된 폐쇄망 리눅스 서버**에 설치하는 과정을 100% 똑같이 모의 훈련하는 풀(Full) 가이드입니다.

Windows의 `C:\` 드라이브에 폴더를 만드는 것이, 리눅스 서버에 USB를 꽂는(`/mnt/c/`) 것과 완벽히 동일한 환경을 제공합니다. 노트북 사양을 고려해 **가장 가벼운 0.5B 모델**을 기준으로 진행합니다.

### 📦 1단계: [온라인] 가상 USB 폴더에 파일 모으기 (Windows 환경)

가장 먼저 Windows 바탕화면이나 C드라이브 최상단에 **`OFFLINE_USB`**라는 폴더를 만듭니다. (경로 예: `C:\OFFLINE_USB`)
인터넷이 연결된 상태에서 아래 파일들을 이 폴더 안에 모두 모아주세요.

**1. AI 모델 파일 (Hugging Face)**
*   **LLM (Qwen 0.5B)**: [이곳](https://huggingface.co/Qwen/Qwen2.5-0.5B-Instruct-GGUF/tree/main)에서 `qwen2.5-0.5b-instruct-q4_k_m.gguf` 파일을 다운로드합니다. (약 400MB)
*   **임베딩 (BGE-M3)**: [이곳](https://huggingface.co/bge-m3-gguf/tree/main)에서 `bge-m3-q4_k_m.gguf` 파일을 다운로드합니다. (약 2GB)

**2. Ollama 리눅스 엔진**
*   [Ollama GitHub Releases](https://github.com/ollama/ollama/releases)에 접속합니다.
*   **`ollama-linux-amd64`** 파일을 다운로드합니다. (Windows용 `.exe`가 아닙니다!)

**3. Dify 소스 및 도커 이미지 추출**
*(※ 노트북에 Docker Desktop이 설치되어 있고 실행 중이어야 합니다.)*
*   [Dify GitHub](https://github.com/langgenius/dify)에서 `Code -> Download ZIP`을 눌러 소스를 받고, `OFFLINE_USB` 폴더 안에 압축을 풉니다. (폴더명이 `dify` 또는 `dify-main`이 됩니다.)
*   Windows 터미널(CMD 또는 PowerShell)을 열고 아래 명령어를 입력해 이미지를 굽습니다.
    ```powershell
    # 압축을 푼 Dify의 docker 폴더로 이동
    cd C:\OFFLINE_USB\dify-main\docker
    
    # 이미지 다운로드
    docker compose pull
    
    # 다운받은 이미지를 하나의 파일로 묶기 (OFFLINE_USB 폴더에 저장)
    docker save -o C:\OFFLINE_USB\dify_images.tar $(docker images -q)
    ```

---

### 🚀 2단계: [오프라인 모의 훈련] WSL 리눅스 터미널 설치

이제 진짜 폐쇄망에 들어왔다고 가정합니다. (원하신다면 노트북 와이파이를 끄셔도 좋습니다.)
**WSL 터미널(Ubuntu 등)**을 실행하고 아래 리눅스 명령어를 순서대로 입력합니다.

**1. 가상 USB 경로로 이동**
```bash
cd /mnt/c/OFFLINE_USB
