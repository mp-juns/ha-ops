# Privacy and Security

Home Assistant configurations can contain highly sensitive data.

## Never commit

- API keys
- provider IDs
- MAC addresses
- internal IP addresses
- SSH usernames
- SSH private keys
- camera URLs
- RTSP URLs
- SSL certificates
- snapshots from personal spaces
- Home Assistant `.storage/`
- database files
- logs

## Recommended practices

- Use `secrets.yaml`
- Commit only `secrets.example.yaml`
- Replace real entity names with generic placeholders
- Avoid uploading real snapshots
- Keep camera files ignored through `.gitignore`
- Rotate any key accidentally pasted into public places
