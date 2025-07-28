FROM parent-java:latest

# Switch to root to install packages
USER root

# Install Python
RUN apk add --no-cache python3

# Create app directory
WORKDIR /app

# Copy our Python script
COPY app/app.py .

# Make the script executable
RUN chmod +x app.py

# Switch back to non-root user for security
USER demouser

# Override the parent's ENTRYPOINT and CMD
ENTRYPOINT []
CMD ["python3", "/app/app.py"]
