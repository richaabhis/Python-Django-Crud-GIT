# Stage 1: Builder
FROM python:3.11 AS builder

# Environment variable to ensure stdout and stderr are flushed
ENV PYTHONUNBUFFERED 1

# Set working directory
WORKDIR /app

# Copy requirements file
COPY requirements.txt /app/

# Create a virtual environment
RUN python -m venv /opt/venv

# Update PATH environment variable
ENV PATH="/opt/venv/bin:$PATH"

# Upgrade pip
RUN pip install --upgrade pip

# Install build dependencies and MySQL client libraries
RUN apt-get update && apt-get install -y \
    build-essential \
    libmariadb-dev-compat \
    libmariadb-dev \
    && apt-get clean

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . /app/

# Stage 2: Final Image
FROM python:3.11-slim

# Install runtime dependencies (libmariadb3)
RUN apt-get update && apt-get install -y \
    libmariadb3 \
    && apt-get clean

# Environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# Set working directory
WORKDIR /app

# Copy virtual environment from builder stage
COPY --from=builder /opt/venv /opt/venv

# Update PATH environment variable
ENV PATH="/opt/venv/bin:$PATH"

# Copy application code
COPY . /app/

# Expose port 8000
EXPOSE 8000

# Entry point
ENTRYPOINT ["python3"]

# Default command
CMD ["manage.py", "runserver", "0.0.0.0:8000"]
