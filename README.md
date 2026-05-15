🚀 폐쇄망 Dify AI 서비스 구축 가이드
본 문서는 외부망과 차단된 On-premise 환경(Rocky Linux 8.10)에서 Dify, Ollama, GPU 인프라를 성공적으로 구축하기 위한 마스터 가이드입니다.

🛠️ 1. 시스템 환경 (Target System)
OS: Rocky Linux 8.10 (x86_64)

GPU: NVIDIA 지원 환경 (Driver 595.71 / CUDA 13.2.1)

핵심 스택: Docker (CE 26.1.3), Ollama, Dify (vMain), BGE-M3 Embedding

📦 2. USB 반입 물자 리스트 (Checklist)
⚠️ 주의: 파일 시스템은 반드시 NTFS로 포맷되어 있어야 합니다. (8GB 이상의 대용량 파일 포함)

A. 인프라 및 의존성
[ ] Rocky-8.10-x86_64-dvd1.iso: 의존성 해결용 로컬 레포지토리 원본

[ ] NVIDIA-Linux-x86_64-595.71.05.run: GPU 드라이버

[ ] cuda_13.2.1_595.58.03_linux.run: CUDA 툴킷

B. Docker 엔진 (docker_rpms 폴더)
[ ] containerd.io, docker-ce, docker-ce-cli, docker-buildx-plugin, docker-compose-plugin (총 5개)

[ ] (선택적) nvidia-container-toolkit 관련 RPM (보험용)

C. 애플리케이션 및 모델
[ ] dify_images.tar: Dify 도커 이미지 통합 팩

[ ] dify-main.zip: Dify 소스코드 및 설정 파일

[ ] ollama-linux-amd64.tar.zst: Ollama 바이너리 (Host 설치용)

[ ] bge-m3-Q4_K_M.gguf: 임베딩 모델

🚀 3. 설치 절차 (Deployment Steps)
STEP 1. 로컬 레포지토리 구축 (의존성 지옥 탈출)
인터넷이 안 되므로 ISO 파일을 서버에 마운트하여 dnf가 내부에서 패키지를 찾도록 설정합니다.

Bash
sudo mkdir -p /mnt/rocky-iso
sudo mount -o loop ./Rocky-8.10-x86_64-dvd1.iso /mnt/rocky-iso

# /etc/yum.repos.d/rocky-local.repo 작성 (AppStream, BaseOS 경로 설정)
STEP 2. GPU 인프라 및 Docker 설치
NVIDIA 드라이버 및 CUDA 설치 (.run 파일 실행)

docker_rpms 폴더 내의 모든 파일을 dnf localinstall로 설치

systemctl enable --now docker로 서비스 기동

STEP 3. Dify 서비스 배포
docker load -i dify_images.tar 명령어로 모든 이미지 로드

/opt/dify-main에 소스 압축 해제 및 .env 설정

docker compose up -d로 10여 개의 컨테이너 동시 기동

STEP 4. AI 모델 엔진 설정 (Ollama)
ollama-linux-amd64.tar.zst 압축 해제 후 /usr에 배치

임베딩 모델(bge-m3) 생성을 위한 Modelfile 작성 및 ollama create 실행

🔍 4. 주요 변경 및 특이사항 (History)
설치 방식 변경: 바이너리(.tgz) 방식에서 OS 안정성과 보안(SELinux)을 고려한 RPM 설치 방식으로 전환.

Docker Compose: 독립 실행 파일 대신 Docker의 공식 Plugin 방식(docker-compose-plugin) 채택. (명령어: docker compose 사용)

의존성 해결: 개별 패키지 반입 대신 Full ISO를 반입하여 현장 에러 변수를 차단.

Ollama 배치: GPU 통신 효율을 위해 Docker 내부가 아닌 Host OS 직접 설치 방식 선택.

💡 5. 문제 해결 (Troubleshooting)
포트 개방: 서비스 접속을 위해 80(Dify), 11434(Ollama) 포트 방화벽 해제 필수.

권한 에러: 설치 과정 중 권한 에러 발생 시 sudo 권한 확인 및 SELinux 상태(sestatus) 점검.

GPU 인식: nvidia-smi 명령어로 드라이버가 정상 작동하는지 확인 후 Ollama 기동.

이 가이드는 영상 엔지니어님의 성공적인 반입과 구축을 기원하며 작성되었습니다. 현장에서 막히는 부분이 생기면 언제든 README.md를 다시 확인하세요! 파이팅입니다! 🚀
