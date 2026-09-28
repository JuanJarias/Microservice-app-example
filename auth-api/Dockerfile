# --- Stage 1: build ---
FROM golang:1.18-alpine AS builder
WORKDIR /app
COPY . .
RUN go mod init github.com/bortizf/microservice-app-example/tree/master/auth-api && \
    go mod tidy && \
    go build -o auth-api .

# --- Stage 2: runtime ---
FROM alpine:latest
WORKDIR /app
COPY --from=builder /app/auth-api .
EXPOSE 8000
ENTRYPOINT ["./auth-api"]