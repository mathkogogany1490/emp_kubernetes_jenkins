# ==========================
# 1️⃣ Builder Stage
# ==========================
FROM node:20-alpine AS builder

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

COPY . .

ARG NEXT_PUBLIC_API_BASE_URL
ENV NEXT_PUBLIC_API_BASE_URL=$NEXT_PUBLIC_API_BASE_URL

RUN npm run build


# ==========================
# 2️⃣ Production Stage
# ==========================
FROM node:20-alpine

WORKDIR /app
ENV NODE_ENV=production

# standalone 모드에서 생성된 결과물만 복사
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/public ./public

EXPOSE 3000

CMD ["node", "server.js"]
