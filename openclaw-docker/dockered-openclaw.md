# OpenClaw Docker Setup 

## 1. Initialize OpenClaw (Required Only Once)

Run the onboarding process:

```bash
docker compose run --rm --no-deps --entrypoint node openclaw-gateway \
  dist/index.js onboard --mode local --no-install-daemon
```

Configure the gateway:

```bash
docker compose run --rm --no-deps --entrypoint node openclaw-gateway \
  dist/index.js config set --batch-json \
  '[{"path":"gateway.mode","value":"local"},{"path":"gateway.bind","value":"lan"}]'
```

These commands generate the required configuration inside the mounted `config/` directory.

---

## 4. Start OpenClaw

```bash
docker compose up -d
```

View logs:

```bash
docker compose logs -f
```

Stop:

```bash
docker compose down
```

Restart:

```bash
docker compose restart
```

Enter shell:
```bash
docker exec -it openclaw-gateway bash
```

---

## Note

When the `config/` directory is mounted and empty, OpenClaw cannot find its configuration and exits with:

```
Missing config.
Run `openclaw setup`
or set gateway.mode=local
(or pass --allow-unconfigured)
```

The onboarding commands create the initial configuration (`openclaw.json`) before the gateway starts.

This initialization only needs to be performed once. After the configuration exists in `./config`, subsequent `docker compose up` commands start normally.