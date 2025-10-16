# ---- Build Stage ----
# Use a specific Node.js version. Alpine is used for a smaller image size.
FROM node:20-alpine AS base

# Set the working directory inside the container.
WORKDIR /app

# ---- Dependencies/Build Stage ----
# This stage installs all dependencies and builds the entire monorepo.
FROM base AS builder

# Copy root package files to leverage Docker cache.
# This assumes you are using npm. If using yarn or pnpm, adjust accordingly.
# Paths are relative to the Dockerfile location, pointing to the monorepo root.
COPY ../../package.json ../../package-lock.json ../../turbo.json ./

# Copy the rest of the monorepo source code.
# This must be done BEFORE npm install for monorepo workspaces to link correctly.
COPY ../.. .

# Install all dependencies and build the monorepo.
# The 'prepare' script in your package.json triggers the build via 'turbo run build'.
RUN npm install

# ---- Final Stage ----
# This stage creates the final, lean production image.
FROM base AS final

# It's a good practice to run as a non-root user for security.
RUN addgroup -S nodejs && adduser -S nodejs -G nodejs
USER nodejs

# Copy only the necessary production dependencies and built files.
# We focus on the specific package we want to run.
WORKDIR /app/packages/observability-mcp

# Copy the package.json for this specific service.
COPY --from=builder /app/packages/observability-mcp/package.json .

# Copy the built bundle from the builder stage.
COPY --from=builder /app/packages/observability-mcp/dist ./dist

# Expose the port your application listens on.
# Replace 8080 with the actual port used by observability-mcp.
EXPOSE 8080

# Command to run the application.
# This runs the bundled javascript file with the --http flag.
CMD ["node", "dist/bundle.js", "--http"]
