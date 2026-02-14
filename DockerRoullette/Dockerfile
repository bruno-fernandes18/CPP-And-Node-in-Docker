# --- STAGE 1: BUILD ---
# Using a full GCC image to compile the C++ source code.
# This avoids installing compilers in the production image.
FROM gcc:latest AS builder
LABEL stage=builder

# Set working directory for compilation
WORKDIR /usr/src/build
# Copy C++ source and compile
COPY ./cpp/rng_engine.cpp .
RUN g++ -O3 -o rng_engine rng_engine.cpp


# --- STAGE 2: RUNTIME ---
# Using a lightweight Alpine image for the Node.js runtime.
# This ensures a small attack surface and high performance.
FROM node:20-alpine AS runner
LABEL maintainer="Seu Nome <Seu Email>"

WORKDIR /app

# Copy the compiled binary from the builder stage
# The Multi-stage build isolates artifacts from build tools.
COPY --from=builder /usr/src/build/rng_engine ./bin/rng_engine
# Install production dependencies only
COPY package*.json ./
RUN npm install --only=production
# Copy application source code
COPY ./src ./src
# Create a non-root user for security (Best Practice)
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
RUN chown -R appuser:appgroup /app
USER appuser

EXPOSE 3000

# Healthcheck to monitor container status
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/api/v1/roll || exit 1

CMD ["node", "src/index.js"]