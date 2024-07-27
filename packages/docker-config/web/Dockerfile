FROM node:18-alpine AS base

# This Dockerfile is copy-pasted into our main docs at /docs/handbook/deploying-with-docker.
# Make sure you update both files!

FROM base AS builder
# Check https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine to understand why libc6-compat might be needed.
RUN apk update
RUN apk add --no-cache libc6-compat
# Set working directory
WORKDIR /app
RUN npm install -g turbo
COPY . .
RUN turbo prune @americaneagle/mecum --docker

# Add lockfile and package.json's of isolated subworkspace
FROM base AS installer
RUN apk update
RUN apk add --no-cache libc6-compat
WORKDIR /app

# First install the dependencies (as they change less often)
COPY .gitignore .gitignore
COPY --from=builder /app/out/json/ .
COPY --from=builder /app/out/package-lock.json ./package-lock.json
RUN npm ci

# Build the project
COPY --from=builder /app/out/full/ .
COPY turbo.json turbo.json

# Added to trigger nextjs standalone mode
ENV BUILD_STANDALONE true

# Uncomment and use build args to enable remote caching
ENV NODE_ENV=development
ENV JWT_SECRET_KEY=${JWT_SECRET_KEY}
ENV MYMECUM_ID=${MYMECUM_ID}
ENV MYMECUM_SECRET=${MYMECUM_SECRET}
ENV NEXTAUTH_URL=${NEXTAUTH_URL}
ENV NEXTAUTH_SECRET=${NEXTAUTH_SECRET}
ENV NEXT_PUBLIC_ALGOLIA_APPLICATION_ID=${NEXT_PUBLIC_ALGOLIA_APPLICATION_ID}
ENV NEXT_PUBLIC_ALGOLIA_INDEX_NAME=${NEXT_PUBLIC_ALGOLIA_INDEX_NAME}
ENV NEXT_PUBLIC_ALGOLIA_SEARCH_ONLY_KEY=${NEXT_PUBLIC_ALGOLIA_SEARCH_ONLY_KEY}
ENV NEXT_PUBLIC_CLOUDINARY_DOMAIN=${NEXT_PUBLIC_CLOUDINARY_DOMAIN}
ENV NEXT_PUBLIC_ENABLE_STELLATE=${NEXT_PUBLIC_ENABLE_STELLATE}
ENV NEXT_PUBLIC_IMAGE_DOMAINS=${NEXT_PUBLIC_IMAGE_DOMAINS}
ENV NEXT_PUBLIC_MYMECUM_DOMAIN=${NEXT_PUBLIC_MYMECUM_DOMAIN}
ENV NEXT_PUBLIC_URL=${NEXT_PUBLIC_URL}
ENV ODR_SECRET=${ODR_SECRET}
ENV UPSTASH_REDIS_ALLOWED_COUNTRIES=${UPSTASH_REDIS_ALLOWED_COUNTRIES}
ENV UPSTASH_REDIS_REST_TOKEN=${UPSTASH_REDIS_REST_TOKEN}
ENV UPSTASH_REDIS_REST_URL=${UPSTASH_REDIS_REST_URL}
ENV WORDPRESS_APPLICATION_PASSWORD=${WORDPRESS_APPLICATION_PASSWORD}
ENV WORDPRESS_APPLICATION_USERNAME=${WORDPRESS_APPLICATION_USERNAME}
ENV WORDPRESS_PREVIEW_SECRET=${WORDPRESS_PREVIEW_SECRET}
ENV WORDPRESS_URL=${WORDPRESS_URL}
ENV NEXT_PUBLIC_SECRET=${NEXT_PUBLIC_SECRET}

RUN npx turbo build --filter=@americaneagle/mecum...

FROM base AS runner
WORKDIR /app

# Don't run production as root
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
USER nextjs

COPY --from=installer /app/apps/web/next.config.js .
COPY --from=installer /app/apps/web/package.json .

# Automatically leverage output traces to reduce image size
# https://nextjs.org/docs/advanced-features/output-file-tracing
COPY --from=installer --chown=nextjs:nodejs /app/apps/web/.next/standalone ./
COPY --from=installer --chown=nextjs:nodejs /app/apps/web/.next/static ./apps/web/.next/static
COPY --from=installer --chown=nextjs:nodejs /app/apps/web/public ./apps/web/public

# RUN chown -R nextjs:nodejs apps/web/server.js


EXPOSE 3000

ENV PORT 3000

# server.js is created by next build from the standalone output
# https://nextjs.org/docs/pages/api-reference/next-config-js/output
# CMD HOSTNAME="0.0.0.0" node apps/web/server.js

CMD node apps/web/server.js
