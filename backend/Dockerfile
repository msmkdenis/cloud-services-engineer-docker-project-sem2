# Сборка
FROM golang:tip-alpine3.22 AS builder

# Рабочая директория
WORKDIR /app

# Копирование файлов
COPY go.mod go.sum ./

# Загрузка зависимостей
RUN go mod download

# Копирование файлов
COPY . .

# Сборка приложения (без зависимостей и уменьшения размера)
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -ldflags="-w -s" -o /go/bin/app ./cmd/api

# Запуск
FROM alpine:3.21.3

# Установка сертификатов CA для HTTPS запросов
RUN apk --no-cache add ca-certificates && \
    update-ca-certificates

# Создаем непривилегированного пользователя
RUN adduser -D -g '' appuser

# Рабочая директория
WORKDIR /app

# Копирование исполняемого файла из builder
COPY --from=builder /go/bin/app /app/momo-store

# Настройка прав доступа
RUN chmod 500 /app/momo-store && \
    chown -R appuser:appuser /app

# Переключение на созданного непривилегированного пользователя
USER appuser

# Порт
EXPOSE 8081

# Запуск приложения
CMD ["/app/momo-store"]