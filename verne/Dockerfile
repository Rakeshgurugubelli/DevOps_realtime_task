# Use the official VerneMQ image as the base image
FROM vernemq/vernemq:latest

# Set the maintainer label
LABEL maintainer="your-email@example.com"

# Environment variables for VerneMQ configuration
ENV DOCKER_VERNEMQ_ALLOW_ANONYMOUS=on
ENV VMQ_ACCEPT_EULA=yes

# Copy your custom configuration files (if any) to the container
# If you have custom vernemq.conf, you can copy it like this:
COPY vernemq.conf /vernemq/etc/vernemq.conf

# Expose required ports
EXPOSE 1883 8888 9001

# Start VerneMQ
CMD ["vernemq", "start"]
