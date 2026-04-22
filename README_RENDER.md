# Deploy to Render

This project is ready for a test Render deploy as a Python Web Service.

## Render service

Create a new **Web Service** from the Git repository.

- Runtime: Python 3
- Build Command: `./build.sh`
- Start Command: `daphne -b 0.0.0.0 -p $PORT zentry.asgi:application`

Render provides the `PORT` variable automatically. The app must bind to `0.0.0.0`.

## Required environment variables

Set these in Render Dashboard -> your service -> Environment:

```env
DEBUG=False
SECRET_KEY=<generate a long random value>
ALLOWED_HOSTS=<your-service>.onrender.com
CSRF_TRUSTED_ORIGINS=https://<your-service>.onrender.com
DATABASE_URL=<internal database URL from Render PostgreSQL>
```

Optional for realtime WebSocket chat:

```env
REDIS_URL=<internal Redis URL>
```

Without `REDIS_URL`, the app uses Django Channels in-memory channel layer. That is okay for a tiny test, but not reliable for multiple instances or production realtime chat.

## Database

Create a Render PostgreSQL database and copy its **Internal Database URL** into `DATABASE_URL`.

## Static and media files

Static files are served by WhiteNoise after `collectstatic`.

Uploaded media files use local filesystem storage. On Render this is ephemeral unless you add persistent storage or move media to S3/Cloudinary. For a test deploy this is acceptable, but uploaded avatars/post images may disappear after redeploys.

## Create admin user

After the first deploy, open Render Shell and run:

```bash
python manage.py createsuperuser
```
