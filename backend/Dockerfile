FROM python:3.11-slim

# =========================
# Environment
# =========================
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONPATH=/app

WORKDIR /app

# =========================
# System Dependencies
# =========================
RUN apt-get update && apt-get install -y build-essential \
    && rm -rf /var/lib/apt/lists/*

# =========================
# Install Python Packages
# =========================
COPY requirements.txt .
RUN pip install --upgrade pip \
    && pip install --no-cache-dir -r requirements.txt

# =========================
# Copy Project
# =========================
COPY . .

# =========================
# 🔥 Static Collect (핵심)
# =========================
RUN python manage.py collectstatic --noinput

# =========================
# Expose
# =========================
EXPOSE 8000

# =========================
# Run Gunicorn
# =========================
CMD ["gunicorn", "config.wsgi:application",
     "--bind", "0.0.0.0:8000",
     "--workers", "3"]
