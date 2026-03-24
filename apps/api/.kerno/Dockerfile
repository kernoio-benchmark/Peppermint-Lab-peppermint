FROM node:18 AS base

# Enable Corepack for Yarn 4.x support
RUN corepack enable

FROM base AS builder

# Set working directory
WORKDIR /app
COPY . .
# Use npx to run turbo instead of global install
RUN npx --yes turbo@latest prune --scope=api --docker


# Add lockfile and package.json's of isolated subworkspace
FROM base AS installer
WORKDIR /app

# First install dependencies (as they change less often)
COPY .gitignore .gitignore
COPY --from=builder /app/out/json/ .
COPY --from=builder /app/out/yarn.lock ./yarn.lock
# Skip postinstall scripts during dependency installation
RUN yarn install --mode=skip-build


# Build the project and its dependencies
COPY --from=builder /app/out/full/ .
COPY turbo.json turbo.json

# Generate Prisma client before building
RUN yarn workspace api prisma generate

# Run the build which will trigger necessary postinstall scripts
RUN yarn turbo run build --filter=api

FROM base AS runner
WORKDIR /app

# Install curl for healthcheck probes
RUN apt-get update && apt-get install -y --no-install-recommends curl && rm -rf /var/lib/apt/lists/*

# Don't run production as root
# RUN addgroup --system --gid 1001 prod
# RUN adduser --system --uid 1001 prod
# USER prod
COPY --from=installer /app .

EXPOSE 8090
CMD ["node", "apps/api/dist/main.js"]
