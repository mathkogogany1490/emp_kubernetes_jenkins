FROM python:3.11-slim

# 환경 변수
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONPATH=/app

# 작업 디렉토리
WORKDIR /app

# 시스템 패키지 설치 (postgres 빌드 대비)
RUN apt-get update && apt-get install -y \
    build-essential \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# requirements 먼저 복사 (Docker layer caching 최적화)
COPY requirements.txt .

RUN pip install --upgrade pip \
    && pip install --no-cache-dir -r requirements.txt

# 소스 복사
COPY . .

# static 파일 수집
RUN python manage.py collectstatic --noinput

# 포트 오픈
EXPOSE 8000

# Gunicorn 실행
CMD ["gunicorn", "config.wsgi:application", \
     "--bind", "0.0.0.0:8000", \
     "--workers", "3", \
     "--timeout", "120"]
