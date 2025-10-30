# Backend Dockerfile for IDURAR ERP CRM
FROM node:20.9.0-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application source
COPY . .

# Expose backend port
EXPOSE 8888

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
  CMD node -e "require('http').get('http://localhost:8888/api', (r) => {process.exit(r.statusCode === 404 ? 0 : 1)})"

# Start the application
CMD ["npm", "start"]
