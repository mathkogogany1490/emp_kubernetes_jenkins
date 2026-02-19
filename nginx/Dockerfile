FROM nginx:alpine

# 기본 설정 삭제 (선택)
RUN rm /etc/nginx/conf.d/default.conf

COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
