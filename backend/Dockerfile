## ------------------------------- Builder Stage ------------------------------ ##
FROM python:3.13-bookworm AS builder

# Install build dependencies only for compiling Python packages
RUN apt-get update && apt-get install --no-install-recommends -y \
        build-essential curl && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Install uv (dependency manager) securely via curl
RUN curl -LsSf https://astral.sh/uv/install.sh | sh

# Set up uv path
ENV PATH="/root/.local/bin:${PATH}"

WORKDIR /app

# Copy dependency files first for better caching
COPY pyproject.toml uv.lock ./

# Sync dependencies in frozen mode for reproducibility
RUN uv sync --frozen --no-dev

## ------------------------------- Production Stage ------------------------------ ##
FROM python:3.13-slim-bookworm AS production

# Create non-root user with fixed UID/GID for security & consistency
RUN useradd --create-home --uid 1001 --shell /bin/bash appuser
USER appuser

WORKDIR /app

# Copy application source
COPY ./src ./src

# Copy virtual environment from builder
COPY --from=builder /app/.venv .venv

# Ensure venv binaries take priority
ENV PATH="/app/.venv/bin:${PATH}"

# Expose default FastAPI port (override at runtime if needed)
EXPOSE 8080

# Start application with uvicorn, allowing PORT override via env var
CMD ["sh", "-c", "adk web --host 0.0.0.0 --port 8080"]
