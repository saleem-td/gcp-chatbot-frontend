FROM python:3.13-slim

# Install build dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    g++ \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY frontend.py .

EXPOSE 8501

CMD ["streamlit", "run", "frontend.py", "--server.address", "0.0.0.0", "--server.port", "8501"]
