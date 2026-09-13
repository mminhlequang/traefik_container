# Traefik dùng chung trên VPS

## Cấu hình Cloudflare DNS challenge

Traefik dùng Cloudflare DNS-01 challenge để cấp chứng chỉ cho
`fastshiphu.com` và `*.fastshiphu.com`.

Domain rút gọn `fastship.hu` được route vào customer app và dùng HTTP-01
challenge, nên không cần mở rộng Cloudflare token sang zone khác.

1. Vào Cloudflare Dashboard, mở **My Profile > API Tokens**.
2. Chọn **Create Token**, sau đó chọn template **Edit zone DNS**.
3. Cấu hình quyền:
   - `Zone` / `DNS` / `Edit`
   - `Zone` / `Zone` / `Read`
4. Trong **Zone Resources**, chọn **Include > Specific zone > fastshiphu.com**.
5. Tạo token và sao chép ngay; Cloudflare chỉ hiển thị giá trị token một lần.
6. Trên VPS, tạo file `.env` cạnh `docker-compose.traefik.yml`:

```dotenv
CF_DNS_API_TOKEN=token_cloudflare_cua_ban
```

Không commit `.env`. File này đã được thêm vào `.gitignore`.

## Khởi chạy / cập nhật Traefik

```bash
docker compose -f docker-compose.traefik.yml up -d
```

Kiểm tra việc cấp chứng chỉ:

```bash
docker logs -f traefik-container
```

Router customer là wildcard fallback: mọi hostname một cấp như
`shop.fastshiphu.com` sẽ đi vào customer. Các router cụ thể như
`admin.fastshiphu.com`, `api.fastshiphu.com`, `pos.fastshiphu.com` và
`posstaff.fastshiphu.com` vẫn được ưu tiên.

Router `fastship-hu` phục vụ `fastship.hu` bằng cùng customer service.

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
