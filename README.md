#  🤖🔬 tinyrobot.science Web UI

Built with [Next.js](https://nextjs.org/) on Heroku.

### Part of a reference suite

| [Terraform config](https://github.com/mars/tinyrobot-science-terraform) | Web UI | [API](https://github.com/mars/tinyrobot-science-api) |
|-----------|------------|---------|
| infrastructure | front-end app (this repo) | backend app |

## Requires

* Heroku
  * [command-line tools (CLI)](https://devcenter.heroku.com/articles/heroku-command-line)
  * [a free account](https://signup.heroku.com)
* [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
* [Node.js](https://nodejs.org)
* [Next.js](https://github.com/zeit/next.js)

## Development

`git clone` this repo to your local machine, and then:

```bash
cd tinyrobot-science-web-ui/
npm install
API_URL=http://127.0.0.1:5001 npm run dev
```

## Mesh deployment

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
