FROM node:20-alpine

WORKDIR /app

COPY app/package*.json ./
RUN npm ci --omit=dev

COPY app/. .

RUN addgroup -S nodeuser \
    && adduser -S nodeuser -G nodeuser \
    && mkdir -p /app/logs \
    && chown -R nodeuser:nodeuser /app

ENV PORT=3000

EXPOSE 3000

USER nodeuser

HEALTHCHECK --interval=30s --timeout=3s --retries=3 --start-period=10s CMD ["node", "-e", "const http=require('http');const port=process.env.PORT||3000;const req=http.get('http://127.0.0.1:'+port+'/health',res=>process.exit(res.statusCode===200?0:1));req.on('error',()=>process.exit(1));"]

CMD ["node", "index.js"]