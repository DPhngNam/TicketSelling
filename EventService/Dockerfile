# Build stage
FROM node:20-alpine AS build

WORKDIR /usr/src/app

# Copy package files for better caching
COPY ./EventService/package*.json ./

# Install all dependencies including dev dependencies
RUN npm install

# Copy the rest of the application code
COPY ./EventService/. .

# Build the application
RUN npm run build

# Production stage
FROM node:20-alpine AS production

ENV NODE_ENV=production

WORKDIR /usr/src/app

# Copy package.json from the build stage
COPY --from=build /usr/src/app/package.json ./

# Install only production dependencies
RUN npm install --only=production && npm cache clean --force

COPY --from=build /usr/src/app/dist ./dist

# copy config files
COPY --from=build /usr/src/app/swagger-config.ts ./swagger-config.ts

# non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nestjs -u 1001 -G nodejs

RUN apk add --no-cache wget

# change ownership of the working directory to the non-root user
RUN chown -R nestjs:nodejs /usr/src/app

# Switch to non-root user
USER nestjs

EXPOSE 3001

HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3001/health || exit 1

ENV NODE_OPTIONS="--max-old-space-size=2048 --max-http-header-size=16384"

CMD ["node", "dist/src/main.js"]