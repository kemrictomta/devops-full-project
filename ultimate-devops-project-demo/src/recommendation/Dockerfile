# Dockerfile (ENHANCED - production-ready)
# Stage 1: Builder
FROM python:3.12-slim-bookworm AS builder

WORKDIR /tmp/build

# Copy requirements first (layer caching)
COPY src/recommendation/requirements.txt .

# Install dependencies in virtual environment
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"
RUN pip install --no-cache-dir --upgrade pip setuptools wheel && \
    pip install --no-cache-dir setuptools
RUN pip install --no-cache-dir -r requirements.txt

# Stage 2: Runtime (only what's needed)
FROM python:3.12-slim-bookworm

WORKDIR /app

# Create non-root user (security)
RUN useradd -m -u 1000 appuser

# Copy only virtual environment (not build tools)
COPY --from=builder --chown=appuser:appuser /opt/venv /opt/venv

# Set environment
ENV PATH="/opt/venv/bin:$PATH" \
    PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    RECOMMENDATION_PORT=9001

# Copy application from actual project
COPY --chown=appuser:appuser src/recommendation/ .

# Install OpenTelemetry auto-instrumentation (from actual project)
RUN pip install --no-cache-dir "setuptools<81"
RUN opentelemetry-bootstrap -a install

# Switch to non-root user
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
    CMD python -c "import socket; socket.create_connection(('localhost', 9001), timeout=2)"

EXPOSE 9001

# Run with unbuffered logging
CMD ["python", "-u", "recommendation_server.py"]