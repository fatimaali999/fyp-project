# Multi-stage Docker build for optimized image size
# Stage 1: Build stage
FROM python:3.10-slim as builder

WORKDIR /app

# Install system dependencies for building
RUN apt-get update && apt-get install -y \
    gcc \
    g++ \
    make \
    libffi-dev \
    libssl-dev \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements and install Python packages
COPY backend/requirements-production.txt .
RUN pip install --no-cache-dir --user -r requirements-production.txt

# Stage 2: Runtime stage
FROM python:3.10-slim

WORKDIR /app

# Install only runtime dependencies
RUN apt-get update && apt-get install -y \
    ffmpeg \
    libsm6 \
    libxext6 \
    libxrender-dev \
    libgomp1 \
    libglib2.0-0 \
    libgl1-mesa-glx \
    && rm -rf /var/lib/apt/lists/*

# Copy Python packages from builder
COPY --from=builder /root/.local /root/.local

# Copy application code from backend folder
COPY backend/ .

# Create necessary directories
RUN mkdir -p uploads test_dataset/videos .cache/whisper

# Make sure scripts are in PATH
ENV PATH=/root/.local/bin:$PATH

# Set Python to run in unbuffered mode
ENV PYTHONUNBUFFERED=1

# Set Whisper cache directory
ENV WHISPER_CACHE_DIR=/app/.cache/whisper

# Expose port (Railway will override this)
EXPOSE 5001

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD python -c "import requests; requests.get('http://localhost:5001/api/test-db', timeout=5)"

# Run the application
CMD ["python", "app.py"]
