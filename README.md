# Traefik dùng chung trên VPS

## Khởi chạy / cập nhật Traefik

```bash
docker compose -f docker-compose.traefik.yml up -d
```

## Thêm hoặc cập nhật service

1. Cho container cần proxy vào Docker network `common_proxy`:

```yaml
services:
  my-app:
    networks: [common_proxy]

networks:
  common_proxy:
    external: true
```

2. Thêm/sửa `router` và `service` trong `dynamic.yml`:

```yaml
http:
  routers:
    my-app:
      rule: "Host(`app.example.com`)"
      entrypoints: [websecure]
      tls:
        certResolver: letsencrypt
      service: my-app

  services:
    my-app:
      loadBalancer:
        servers:
          - url: "http://TEN_CONTAINER:PORT_NOI_BO"
```

Ví dụ: `http://my-api:8000`. Dùng **tên container** và **port nội bộ**, không dùng port publish ra VPS.

Lưu file là Traefik tự reload. Kiểm tra:

```bash
docker logs -f traefik-container
```

DNS của domain/subdomain phải trỏ về IP VPS; firewall cần mở port `80` và `443`.
