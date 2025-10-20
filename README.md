# EZproxy Redirect Service (Docker + Nginx)

This project provides an HTTP/HTTPS redirect service to preserve legacy EZproxy links after migration to a hosted EZproxy or OCLC IDM environment.

## Purpose
Redirect legacy URLs such as:
```
http://old-ezproxy.example.edu/login?url=https://resource.com/article
http://old-ezproxy.example.edu:2048/login?url=https://resource.com/article
```
To new hosted EZproxy format:
```
https://example.idm.oclc.org/login?url=https://resource.com/article
```

## Requirements
- Docker and Docker Compose
- Public IP
- Host networking or published ports (80, 443, 2048)
- Access to Certbot for SSL

## Deployment

```bash
git clone https://github.com/your-org/ezproxy-redirect.git
cd ezproxy-redirect
docker compose up -d certbot
docker compose up -d nginx
```

## SSL Certificate (Certbot)

```bash
docker compose exec certbot certbot certonly   --webroot -w /var/www/certbot   -d example-ezproxy-redirect.examplehost.com   --email admin@example.org --agree-tos --no-eff-email
```

Update `nginx/conf.d/redirect.conf` with certificate paths, then reload:

```bash
docker compose exec nginx nginx -t
docker compose exec nginx nginx -s reload
```

## Testing

```bash
curl -I "http://example-ezproxy-redirect.examplehost.com/login?url=https://www.jstor.org/stable/1234"
curl -I "http://example-ezproxy-redirect.examplehost.com:2048/login?url=https://muse.jhu.edu/article/5678"
curl -I "https://example-ezproxy-redirect.examplehost.com/login?url=https://ebookcentral.proquest.com/..."
```

## Auto-Renew (Cron)

```bash
17 2 * * * docker compose exec -T certbot certbot renew --quiet && docker compose exec -T nginx nginx -s reload
```

## Troubleshooting

| Issue | Action |
|-------|--------|
| Connection refused | Check `docker compose ps` |
| Cert mismatch | Re-run certbot |
| Restart loop | `docker compose logs nginx` |

## Cutover to Real Hostname

1. Update DNS → server IP  
2. Issue new certificate  
3. Update `server_name` in `redirect.conf`  
4. Reload Nginx

## Summary

| Feature | Status |
|---------|--------|
| Legacy EZproxy redirect | Yes |
| Port 80 / 2048 / 443 support | Yes |
| HTTPS | Yes |
| Cert auto-renew | Yes |
