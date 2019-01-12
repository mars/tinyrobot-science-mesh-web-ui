#  🚧🔬 tinyrobot.science "Kong 1.0 Mesh" experiment

*Based on [this microservices example app](https://github.com/mars/tinyrobot-science-web-ui).*

Let's discover how Kong's new sidecar deployment style integrates with Heroku.

## Results

Kong's actual mesh networking (service-to-service, mutual TLS) features are [facilitated by changing `iptables`](https://docs.konghq.com/1.0.x/streams-and-service-mesh/#step-7-configure-transparent-proxying-rules) configuration on the hosts, which is not possible on Heroku.

Even without the true mesh networking, deploying Kong sidecar-style does still provide some benefits:

- less latency than a separate Kong proxy app (the proxy-backend connection is on localhost)
- Kong cluster inherently scales with the app's dyno scaling

## Open questions

Can a single Kong controller operate many apps worth of Kong sidecars?
  - Do backend `PORT` values conflict?
  - Does the [`KONG_ORIGINS` config](https://docs.konghq.com/1.0.x/streams-and-service-mesh/#the-origins-configuration-option) solve this? (Setting hostnames into config vars is gross 😣)

## Deployment

First, [deploy a Kong app](https://github.com/heroku/heroku-kong) as the mesh controller.

Then, deploy this app:

```bash
heroku create tinyrobot-science-mesh-web-ui
heroku buildpacks:add heroku/nodejs
heroku buildpacks:add heroku-community/kong
heroku buildpacks:add https://github.com/danp/heroku-buildpack-runit

# Attach the ID of the mesh controller's Postgres add-on.
heroku addons:attach postgresql-cylindrical-98791

git push heroku master
```

Then, create the mesh service & route:

(Note: port 5000 is the local listener set in bin/start-app)

```bash
## Create mesh service
curl -X "PUT" "https://tinyrobot-science-mesh-control.herokuapp.com/kong-admin/services/web-ui" \
     -H 'apikey: xxxxx' \
     -H 'Content-Type: application/json; charset=utf-8' \
     -d $'{
  "url": "http://127.0.0.1:5000"
}'

## Create mesh route
curl -X "POST" "https://tinyrobot-science-mesh-control.herokuapp.com/kong-admin/services/web-ui/routes" \
     -H 'apikey: xxxxx' \
     -H 'Content-Type: application/json; charset=utf-8' \
     -d $'{
  "name": "web-ui",
  "hosts": [
    "tinyrobot-science-mesh-web-ui.herokuapp.com"
  ],
  "protocols": [
    "https"
  ]
}'

## Add the current app name as the origin for the web-ui services
heroku config:set KONG_ORIGINS=https://tinyrobot-science-mesh-web-ui.herokuapp.com:443=http://localhost:5000
```

✨ Visit the app's URL `https://tinyrobot-science-mesh-web-ui.herokuapp.com/` in a web browser.
