🚀 Dify AI Service Deployment Guide (Air-Gapped)
본 프로젝트는 외부망과 완전히 차단된 폐쇄망(On-premise) 환경에서 Dify, Ollama, 그리고 GPU 인프라를 구축하기 위한 마스터 가이드입니다. 신한DS 인프라 환경(Rocky Linux 8.10)에 최적화되어 있습니다.

🛠️ 1. 시스템 환경 (System Environment)
OS: Rocky Linux 8.10 (x86_64)

Kernel: 4.18.0-553.el8_10.x86_64 (추천)

GPU: NVIDIA 지원 환경

Driver: v595.71

CUDA: v13.2.1

Stack:

Docker: CE 26.1.3

Orchestration: Docker Compose (Plugin)

LLM Engine: Ollama

Embedding: BGE-M3 (GGUF)

📦 2. USB 반입 물자 리스트 (Checklist)
⚠️ 중요: USB 파일 시스템은 반드시 NTFS로 포맷되어야 합니다. (ISO 및 이미지 파일이 4GB를 초과함)

A. 인프라 및 의존성
[ ] Rocky-8.10-x86_64-dvd1.iso: 의존성 해결용 로컬 레포지토리 원본

[ ] NVIDIA-Linux-x86_64-595.71.05.run: GPU 드라이버 설치 파일

[ ] cuda_13.2.1_595.58.03_linux.run: CUDA 툴킷 설치 파일

B. Docker 엔진 (docker_rpms/ 폴더)
[ ] containerd.io-1.6.31-3.1.el8.x86_64.rpm

[ ] docker-ce-26.1.3-1.el8.x86_64.rpm

[ ] docker-ce-cli-26.1.3-1.el8.x86_64.rpm

[ ] docker-buildx-plugin-0.14.0-1.el8.x86_64.rpm

[ ] docker-compose-plugin-2.27.0-1.el8.x86_64.rpm

[ ] (Optional) nvidia-container-toolkit 관련 RPM 세트

C. 애플리케이션 및 모델
[ ] dify_images.tar: Dify 서비스 운영에 필요한 모든 도커 이미지 팩

[ ] dify-main.zip: Dify 소스코드 및 환경 설정 파일

[ ] ollama-linux-amd64.tar.zst: Ollama 바이너리 (Host 직접 설치용)

[ ] bge-m3-Q4_K_M.gguf: RAG용 고성능 임베딩 모델

🚀 3. 설치 절차 (Installation Steps)
STEP 1: 로컬 레포지토리 구성
인터넷이 차단된 환경에서 패키지 의존성을 해결하기 위해 ISO 파일을 활용합니다.

Bash
# ISO 마운트
sudo mkdir -p /mnt/rocky-iso
sudo mount -o loop ./Rocky-8.10-x86_64-dvd1.iso /mnt/rocky-iso

# 기존 레포지토리 백업
sudo mkdir -p /etc/yum.repos.d/backup
sudo mv /etc/yum.repos.d/*.repo /etc/yum.repos.d/backup/

# 로컬 레포 설정 작성
sudo vi /etc/yum.repos.d/rocky-local.repo
rocky-local.repo 내용:

Ini, TOML
[Local-BaseOS]
name=Rocky Local BaseOS
baseurl=file:///mnt/rocky-iso/BaseOS
enabled=1
gpgcheck=0

[Local-AppStream]
name=Rocky Local AppStream
baseurl=file:///mnt/rocky-iso/AppStream
enabled=1
gpgcheck=0
STEP 2: 인프라 및 Docker 설치
Bash
# Docker RPM 설치 (의존성은 ISO에서 자동 해결)
cd ./docker_rpms
sudo dnf localinstall -y *.rpm
sudo systemctl enable --now docker

# GPU 드라이버 및 CUDA 설치
chmod +x NVIDIA-Linux-x86_64-595.71.05.run cuda_13.2.1_595.58.03_linux.run
sudo ./NVIDIA-Linux-x86_64-595.71.05.run
sudo ./cuda_13.2.1_595.58.03_linux.run
STEP 3: Dify 서비스 기동
Bash
# 이미지 로드
sudo docker load -i dify_images.tar

# 소스 배치 및 설정
sudo unzip dify-main.zip -d /opt/
cd /opt/dify-main/docker
cp .env.example .env

# 서비스 실행
sudo docker compose up -d
STEP 4: Ollama 및 임베딩 모델 설정
Bash
# Ollama 설치
sudo tar -C /usr -jxvf ollama-linux-amd64.tar.zst

# 서비스 실행 (백그라운드)
ollama serve &

# 임베딩 모델 생성용 Modelfile 작성
echo "FROM ./bge-m3-Q4_K_M.gguf" > Modelfile_bge
ollama create bge-m3 -f Modelfile_bge
🔍 4. 주요 변경 사항 (History)
설치 표준화: 바이너리 설치 대신 OS 정규 패키지(RPM) 방식을 채택하여 SELinux 보안 정책 충돌 방지.

Docker Compose: 독립 바이너리(v1)에서 Docker 공식 플러그인(v2)으로 전환하여 명령어를 docker compose로 단일화.

Ollama 최적화: GPU 통신 효율 및 관리 편의성을 위해 Docker 컨테이너 외부(Host OS)에 직접 설치.

안정성 강화: 현장 의존성 에러를 원천 차단하기 위해 Full ISO 기반의 로컬 저장소 구축 전략 도입.

💡 5. 문제 해결 (Troubleshooting)
방화벽 설정: 외부 브라우저 접속을 위해 포트 개방이 필요합니다.

sudo firewall-cmd --permanent --add-port=80/tcp (Dify)

sudo firewall-cmd --permanent --add-port=11434/tcp (Ollama)

sudo firewall-cmd --reload

SELinux 권한: 컨테이너가 볼륨에 접근하지 못할 경우 sestatus를 확인하거나 대상 폴더의 보안 문맥을 수정하세요.

GPU 인식 불가: nvidia-smi 명령어 결과가 정상인지 확인하고, Ollama 서비스가 드라이버를 로드했는지 체크하세요.

Maintainer: 조영상 (Backend Engineer)
