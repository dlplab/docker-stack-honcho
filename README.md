# docker-stack-honcho

[Honcho](https://github.com/plastic-labs/honcho) v3.4.0 (API + deriver + Redis), deployed as a Portainer stack.
Uses the external PostgreSQL 18/pgvector stack `dlplab/docker-stack-postgres`.

## Prerequisites

- Postgres stack running; DB/role `honcho` with extensions `vector`, `pg_trgm`, `btree_gin`
  created by the superuser (the `honcho` role cannot create `vector`).
- External networks `postgres` and `frontend` (Traefik).

## Deploy

Portainer > Stacks > Add stack > Repository: `dlplab/docker-stack-honcho`, compose path `stack.yaml`.
Stack env vars: see `.env.example` (`FQDN_HONCHO`, `DB_CONNECTION_URI`, `REDIS_PASSWORD`,
`AUTH_JWT_SECRET`, one `LLM_*_API_KEY`). Use hex passwords (`openssl rand -hex 32`) — they go into URLs.

The API container runs migrations on start (advisory-locked); the deriver waits for the API to be healthy.

## Auth / client key

Auth is on by default. Generate an admin JWT (signed with `AUTH_JWT_SECRET`):

```sh
docker exec honcho-api /app/.venv/bin/python scripts/generate_jwt.py --admin
```

Scoped keys: `--workspace <ws> [--peer <p>] --expires 1y`. Rotating `AUTH_JWT_SECRET` invalidates all keys.

Hermes: `baseUrl=https://<FQDN_HONCHO>`, `apiKey=<JWT>`.
