## Description

This example shows you [NPMpro](https://github.com/TheBlackBush/NPMpro), a fork of NPMplus and nginx-proxy-manager with the CrowdSec bouncer built in.

This example contains multiple containers:
* npmpro : The NPMpro container, writing its nginx logs to `/opt/npmpro/nginx/logs`
* crowdsec : Reads the NPMpro logs from the mounted folder (collection `TheBlackBush/npmpro`) and runs AppSec for the NPMpro bouncer

You also need to create an API key for the bouncer in CrowdSec and configure NPMpro:

```bash
sed -i 's|"TZ=.*"|"TZ=<your-timezone>"|g' compose.yaml # set your time zone here, for example Europe/Berlin
docker compose up -d
docker exec crowdsec cscli hub update && docker exec crowdsec cscli hub upgrade # make sure the latest parser versions are installed
docker restart crowdsec
docker exec crowdsec cscli bouncers add npmpro -o raw
sed -i "s|^API_KEY=.*|API_KEY=<key shown by the command above>|" /opt/npmpro/crowdsec/crowdsec.conf # set the API key here
sed -i "s|^ENABLED=.*|ENABLED=true|" /opt/npmpro/crowdsec/crowdsec.conf
docker restart npmpro
```

`API_URL` (`http://127.0.0.1:8080`) and `APPSEC_URL` (`http://127.0.0.1:7422`) in `crowdsec.conf` already match this example. If you do not run AppSec, clear `APPSEC_URL`, otherwise the bouncer blocks every request while AppSec cannot be reached.

Check that it works: `docker exec crowdsec cscli metrics` should show the access log lines as parsed, `docker exec crowdsec cscli bouncers list` should list `npmpro`, and a request to `/.env` on one of your proxy hosts should be answered with 403. The NPMpro web UI shows the active bans under Monitoring, CrowdSec. More details: https://github.com/TheBlackBush/NPMpro/wiki/CrowdSec

**Prerequisites:** [Docker](https://docs.docker.com/engine/install) / [Docker Compose](https://docs.docker.com/compose/install)
