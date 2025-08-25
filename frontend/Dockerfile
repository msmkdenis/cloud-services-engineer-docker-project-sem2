# Сборка
FROM node:22-alpine AS builder

# Переменная для сборки
ENV NODE_OPTIONS=--openssl-legacy-provider

# Добавляем пользователя и группу
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Рабочая директория
WORKDIR /app

# Копирование файлов
COPY package.json package-lock.json ./

# Установка зависимостей
RUN npm ci --no-audit --no-fund

# Копирование файлов
COPY . .

# Сборка
RUN npm run build && \
    chown -R appuser:appgroup /app/dist

# Запуск
FROM nginx:1.29-alpine

# Добавляем пользователя и группу
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Создаем директорию приложения
RUN mkdir -p /usr/share/nginx/html/momo-store

# Копирование собранного приложения из этапа сборки
COPY --from=builder --chown=appuser:appgroup /app/dist/ /usr/share/nginx/html/momo-store/

# Копирование конфигурации для nginx
RUN mkdir -p /etc/nginx/conf.d
COPY --chown=appuser:appgroup nginx.conf /etc/nginx/conf.d/default.conf

# Установка прав доступа для директорий
RUN chown -R appuser:appgroup /var/cache/nginx && \
    chown -R appuser:appgroup /var/log/nginx && \
    chown -R appuser:appgroup /etc/nginx/conf.d && \
    touch /var/run/nginx.pid && \
    chown -R appuser:appgroup /var/run/nginx.pid && \
    chmod -R 755 /usr/share/nginx/html

# Обновление пакетов trivy
RUN apk update && apk upgrade

# Переключение на созданного непривилегированного пользователя
USER appuser

# Порт
EXPOSE 80

# Запуск nginx
CMD ["nginx", "-g", "daemon off;"]