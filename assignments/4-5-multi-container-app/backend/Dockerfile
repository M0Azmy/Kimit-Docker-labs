FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
COPY tsconfig.json ./
RUN npm install

COPY src/ ./src/
RUN npm run build

FROM node:18-alpine AS production

WORKDIR /app
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodeuser -u 1001

COPY package*.json ./
RUN npm install --only=production

COPY --from=builder --chown=nodeuser:nodejs /app/dist/ ./dist/

USER nodeuser

EXPOSE 3001
CMD ["node", "dist/index.js"]