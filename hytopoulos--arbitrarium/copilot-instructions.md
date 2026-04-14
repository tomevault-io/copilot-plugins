## arbitrarium

> All debugging of frontend/ and backend/ must take place inside the docker containers.


All debugging of frontend/ and backend/ must take place inside the docker containers.

You may test on the backend like so:
docker compose exec backend python manage.py shell -c "from coreapp.models import Frame; frame=Frame.from_framenet(xxx);"

Always check docker logs for build or runtime errors

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hytopoulos) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
