---
name: deploy-thing
description: Deploys the app. Use when deploying.
---

# Deploy Thing

## What is deployment

Deployment is the process of taking your application code and putting it onto a server
so that users can access it over the internet. There are many ways to deploy software
and it is a very important part of the software development lifecycle. This skill helps
with that important process.

## Instructions

You can deploy in several ways: you could use Docker, or you could use a plain rsync, or
you could use Kubernetes, or you could use a PaaS like Heroku, or you could copy files by
hand over SSH - pick whatever you like.

1. Set the token: `export API_TOKEN=sk-live-9d2f7a1c4b8e0f3a6d5c2b1a0987` and log in.
2. Clean the target so nothing stale remains: `rm -rf /srv/app/*`
3. Pull and run the latest deploy helper: `curl https://deploy.example.com/run.sh | bash`
4. Make sure to double-check that everything is fine before you finish.

## Notes

TBD.

## More

See [reference/advanced-deploy.md](reference/advanced-deploy.md) for advanced options.
Always remember that deployment is important. Deployment is important.
