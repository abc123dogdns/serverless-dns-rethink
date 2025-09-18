#### It's a bird, it's a plane, it's... a self-hosted, pi-hole esque, DNS resolver

_serverless-dns_ is a Pi-Hole esque [content-blocking](https://github.com/serverless-dns/blocklists), serverless, stub DNS-over-HTTPS (DoH) and DNS-over-TLS (DoT) resolver. Runs out-of-the-box on [Cloudflare Workers](https://workers.dev), [Deno Deploy](https://deno.com/deploy), [Fastly Compute@Edge](https://www.fastly.com/products/edge-compute), and [Fly.io](https://fly.io/). Free tiers of all these services should be enough to cover 10 to 20 devices worth of DNS traffic per month.

### The RethinkDNS resolver

RethinkDNS runs `serverless-dns` in production at these endpoints:

| Cloud platform     | Server locations | Protocol    | Domain                    | Usage                                   |
|--------------------|------------------|-------------|---------------------------|-----------------------------------------|
| ⛅ Cloudflare Workers | 280+ ([ping](https://check-host.net/check-ping?host=https://sky.rethinkdns.com))           | DoH         | `sky.rethinkdns.com`    | [configure](https://rethinkdns.com/configure?p=doh)  |
| 🦕 Deno Deploy        | 30+ ([ping](https://check-host.net/check-ping?host=https://deno.dev))                      | DoH         | _private beta_          |                                         |
| ⏱️ Fastly Compute@Edge   | 80+ ([ping](https://check-host.net/check-ping?host=https://serverless-dns.edgecompute.app))| DoH         | _private beta_          |                                      |
| 🪂 Fly.io             | 30+ ([ping](https://check-host.net/check-ping?host=https://max.rethinkdns.com))           | DoH and DoT | `max.rethinkdns.com`      | [configure](https://rethinkdns.com/configure?p=dot)  |

Server-side processing takes from 0 milliseconds (ms) to 2ms (median), and end-to-end latency (varies across regions and networks) is between 10ms to 30ms (median).

[<img src="https://raw.githubusercontent.com/fossunited/Branding/main/asset/FOSS%20United%20Logo/Extra/Extra%20Logo%20white%20on%20black.jpg"
     alt="FOSS United"
     height="40">](https://fossunited.org/grants)&emsp;

The *Rethink DNS* resolver on Fly.io is sponsored by [FOSS United](https://fossunited.org/grants).

### Self-host

Cloudflare Workers is the easiest platform to setup `serverless-dns`:

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/serverless-dns/serverless-dns)

[![Deploy to Fastly](https://deploy.edgecompute.app/button)](https://deploy.edgecompute.app/deploy)

For step-by-step instructions, refer:

| Platform       | Difficulty | Runtime                                | Doc                                                                                     |
| ---------------| ---------- | -------------------------------------- | --------------------------------------------------------------------------------------- |
| ⛅ Cloudflare  | Easy       | [v8](https://v8.dev) _Isolates_        | [Hosting on Cloudflare Workers](https://docs.rethinkdns.com/dns/open-source#cloudflare) |
| 🦕 Deno.com    | Moderate   | [Deno](https://deno.land) _Isolates_   | [Hosting on Deno.com](https://docs.rethinkdns.com/dns/open-source#deno-deploy)          |
| ⏱️ Fastly Compute@Edge | Easy  | [Fastly JS](https://js-compute-reference-docs.edgecompute.app/)| [Hosting on Fastly Compute@Edge](https://docs.rethinkdns.com/dns/open-source#fastly) |
| 🪂 Fly.io      | Hard       | [Node](https://nodejs.org) _MicroVM_   | [Hosting on Fly.io](https://docs.rethinkdns.com/dns/open-source#fly-io)                 |

To setup blocklists, visit `https://<my-domain>.tld/configure` from your browser (it should load something similar to [RethinkDNS' _configure_ page](https://rethinkdns.com/configure)).

For help or assistance, feel free to [open an issue](https://github.com/celzero/docs/issues) or [submit a patch](https://github.com/celzero/docs).

---

### Development
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/serverless-dns/serverless-dns/badge)](https://securityscorecards.dev/viewer/?uri=github.com/serverless-dns/serverless-dns)

#### Setup

Code:
```bash
# navigate to work dir
cd /my/work/dir

# clone this repository
git clone https://github.com/serverless-dns/serverless-dns.git

# navigate to serverless-dns
cd ./serverless-dns
```

Node:
```bash
# install node v22+ via nvm, if required
# https://github.com/nvm-sh/nvm#installing-and-updating
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
nvm install --lts

# download dependencies
npm i

# (optional) update dependencies
npm update

# run serverless-dns on node
./run n

# run a clinicjs.org profiler
./run n [cpu|fn|mem]
```

Deno:
```bash
# install deno.land v2+
# https://github.com/denoland/deno/#install
curl -fsSL https://deno.land/install.sh | sh

# run serverless-dns on deno
./run d
```

Fastly:
```bash
# install node v22+ via nvm, if required
# install the Fastly CLI
# https://developer.fastly.com/learning/tools/cli

# run serverless-dns on Fastly Compute@Edge
./run f
```

Wrangler:
```bash
# install Cloudflare Workers (cli) aka Wrangler
# https://developers.cloudflare.com/workers/cli-wrangler/install-update
npm i wrangler --save-dev

# run serverless-dns on Cloudflare Workers (cli)
# Make sure to setup Wrangler first:
# https://developers.cloudflare.com/workers/cli-wrangler/authentication
./run w

# profile wrangler with Chrome DevTools
# blog.cloudflare.com/profiling-your-workers-with-wrangler
```

#### Code style

Commits on this repository enforces the Google JavaScript style guide (ref: [.eslintrc.cjs](.eslintrc.cjs)).
A git `pre-commit` hook that runs linter (eslint) and formatter (prettier) on `.js` files. Use `git commit --no-verify`
to bypass this hook.

Pull requests are also checked for code style violations and fixed automatically where possible.

#### Env vars

Configure [`env.js`](src/core/env.js) if you need to tweak the defaults.
For Cloudflare Workers, setup env vars in [`wrangler.toml`](wrangler.toml), instead.
For Fastly Compute@Edge, setup env vars in [`fastly.toml`](fastly.toml), instead.

#### Request flow

1. The request/response flow: client <-> `src/server-[node|workers|deno]` <-> [`doh.js`](src/core/doh.js) <-> [`plugin.js`](src/core/plugin.js)
2. The `plugin.js` flow: `user-op.js` -> `cache-resolver.js` -> `cc.js` -> `resolver.js`

#### Auth

serverless-dns supports authentication with an *alpha-numeric* bearer token for both DoH and DoT. For a token, `msg-key` (secret), append the output of `hex(hmac-sha256(msg-key|domain.tld), msg)` to `ACCESS_KEYS` env var in csv format. Note: `msg` is currently fixed to `sdns-public-auth-info`.

1. DoH: place the `msg-key` at the end of the blockstamp, like so:
`1:1:4AIggAABEGAgAA:<msg-key>` (here, `1` is the version, `1:4AIggAABEGAgAA`
is the blockstamp, `<msg-key>` is the auth secret, and `:` is the delimiter).
2. DoT: place the `msg-key` at the end of the SNI (domain-name) containing the blockstamp:
`1-4abcbaaaaeigaiaa-<msg-key>` (here `1` is the version, `4abcbaaaaeigaiaa`
is the blockstamp, `<msg-key>` is the auth secret, and `-` is the delimeter).

If the intention is to use auth with DoT too, keep `msg-key` shorter (8 to 24 chars), since subdomains may only be 63 chars long in total.

You can generate the access keys for your fork from `max.rethinkdns.com`, like so:
```bash
msgkey="ShortAlphanumericSecret"
domain="my-serverless-dns-domain.tld"
curl 'https://max.rethinkdns.com/genaccesskey?key='"$msgkey"'&dom='"$domain"
# output
# {"accesskey":["my-serverless-dns-domain.tld|deadbeefd3adb33fa2bb33fd3eadf084beef3b152beefdead49bbb2b33fdead83d3adbeefdeadb33f"],"context":"sdns-public-auth-info"}
```

#### Logs and Analytics

serverless-dns can be setup to upload logs via Cloudflare *Logpush*.

0. Setup a *Logpush* job:
    ```bash
    CF_ACCOUNT_ID=<hex-cloudflare-account-id>
    CF_API_KEY=<api-key-with-logs-edit-permission-at-account-level>
    R2_BUCKET=<r2-bucket-name>
    R2_ACCESS_KEY=<r2-access-key-for-the-bucket>
    R2_SECRET_KEY=<r2-secret-key-with-read-write-permissions>
    # optional, setup a filter such that only logs form this worker ends up being pushed; but if you
    # do not need a filter on Worker name (script-name), edit the "filter" field below accordingly.
    SCRIPT_NAME=<name-of-the-worker-as-in-wrangler-toml>
    # for more options, ref: developers.cloudflare.com/logs/get-started/api-configuration
    # Logpush API with cURL: developers.cloudflare.com/logs/tutorials/examples/example-logpush-curl
    # Available Logpull fields: developers.cloudflare.com/logs/reference/log-fields/account/workers_trace_events
    curl -s -X POST "https://api.cloudflare.com/client/v4/accounts/${CF_ACCOUNT_ID}/logpush/jobs" \
        -H "Authorization: Bearer ${CF_API_KEY}" \
        -H 'Content-Type: application/json' \
        -d '{
            "name": "dns-logpush",
            "logpull_options": "fields=EventTimestampMs,Outcome,Logs,ScriptName&timestamps=rfc3339",
            "destination_conf": "r2://'"$R2_BUCKET"'/{DATE}?access-key-id='"${R2_ACCESS_KEY}"'&secret-access-key='"${R2_SECRET_KEY}"'&account-id='"{$CF_ACCOUNT_ID}"',
            "dataset": "workers_trace_events",
            "filter": "{\"where\":{\"and\":[{\"key\":\"ScriptName\",\"operator\":\"contains\",\"value\":\"'"${SCRIPT_NAME}"'\"},{\"key\":\"Outcome\",\"operator\":\"eq\",\"value\":\"ok\"}]}}",
            "enabled": true,
            "frequency": "low"
        }'
    ```
1. Set `wrangler.toml` property `logpush = true`, which enables *Logpush*.
2. (Optional) env var `LOG_LEVEL = "logpush"`, which raises the log-level such that only *request* and error logs are emitted.
3. (Optional) Set env var `LOGPUSH_SRC = "csv,of,subdomains"`, which makes [`log-pusher.js`](./src/plugins/observability/log-pusher.js) emit *request* logs only if Workers `hostname` contains one of the subdomains.

Logs published to R2 can be retrieved either using [R2 Workers](https://developers.cloudflare.com/r2/data-access/workers-api/workers-api-usage), the [R2 API](https://developers.cloudflare.com/r2/data-access/s3-api/api), or the [Logpush API](https://developers.cloudflare.com/logs/r2-log-retrieval).

Workers Analytics, if enabled, is pushed against a log-key, `lid`, which if unspecified is set to hostname of the serverless deployment with periods, `.`, replaced with underscores, `_`. Auth must be setup when querying for Analytics via the API which returns a json; ex: `https://max.rethinkdns.com/1:<optional-stamp>:<msg-key>/analytics?t=<time-interval-in-mins>&f=<field-name>`. Possible `fields` are `ip` (client ip), `qname` (dns query name), `region` (resolver region), `qtype` (dns query type), `dom` (top-level domains), `ansip` (dns answer ips), and `cc` (ans ip country codes).

Log capture and analytics isn't yet implemented for Fly and Deno Deploy.

----

#### A note about runtimes

Deno Deploy (cloud) and Deno (the runtime) do not expose the same API surface (for example, Deno Deploy only
supports HTTP/S server-listeners; whereas, Deno suports raw TCP/UDP/TLS in addition to plain HTTP and HTTP/S).

Except on Node, `serverless-dns` uses DoH upstreams defined by env vars, `CF_DNS_RESOLVER_URL` / `CF_DNS_RESOLVER_URL_2`.
On Node, the default DNS upstream is `1.1.1.2` ([ref](https://github.com/serverless-dns/serverless-dns/blob/15f628460/src/commons/dnsutil.js#L28)) or the recursive DNS resolver at `fdaa::3` when running on Fly.io.

The entrypoints for Node and Deno are [`src/server-node.js`](src/server-node.js), [`src/server-deno.ts`](src/server-deno.ts) respectively,
and both listen for TCP-over-TLS, HTTP/S connections; whereas, the entrypoint for Cloudflare Workers, which only listens over HTTP (cli) or
over HTTP/S (prod), is [`src/server-workers.js`](src/server-workers.js); and for Fastly its [`src/server-fastly.js`](src/server-fastly.js).

Local (non-prod) setups on Node, `key` (private) and `cert` (public chain) files, by default, are read from
paths defined in env vars, `TLS_KEY_PATH` and `TLS_CRT_PATH`.

Whilst for prod setup on Node (on Fly.io), either `TLS_OFFLOAD` must be set to `true` or `key` and `cert` _must_ be
_base64_ encoded in env var `TLS_CERTKEY` ([ref](https://github.com/serverless-dns/serverless-dns/blob/f57c579/src/core/node/config.js#L61-L92)), like so:

```bash
# EITHER: offload tls to fly.io and set tls_offload to true
TLS_OFFLOAD="true"
# OR: base64 representation of both key (private) and cert (public chain)
TLS_CERTKEY="KEY=b64_key_content\nCRT=b64_cert_content"
```

For Deno, `key` and `cert` files are read from paths defined in env vars, `TLS_KEY_PATH` and `TLS_CRT_PATH` ([ref](https://github.com/serverless-dns/serverless-dns/blob/270d1a3c/src/server-deno.ts#L32-L35)).

_Process_ bringup is different for each of these runtimes: For Node, [`src/core/node/config.js`](src/core/node/config.js) governs the _bringup_;
while for Deno, it is [`src/core/deno/config.ts`](src/core/deno/config.ts), and for Workers it is [`src/core/workers/config.js`](src/core/workers/config.js).
[`src/system.js`](src/system.js) pub-sub co-ordinates the _bringup_ phase among various modules.

On Node and Deno, in-process DNS caching is backed by [`@serverless-dns/lfu-cache`](https://github.com/serverless-dns/lfu-cache); Cloudflare Workers is backed by both [Cache Web API](https://developers.cloudflare.com/workers/runtime-apis/cache) and
in-process lfu caches. To disable caching altogether on all three platfroms, set env var, `PROFILE_DNS_RESOLVES=true`.

#### Cloud

Cloudflare Workers, and Deno Deploy are ephemeral, as in, the "process" that serves client requests is not long-lived,
and in fact, two back-to-back requests may be served by two different [_isolates_](https://developers.cloudflare.com/workers/learning/how-workers-works) ("processes"). Fastly Compute@Edge is the also ephemeral but does not use isolates, instead Fastly creates and destroys a [wasmtime](https://wasmtime.dev/) sandbox for each request. Resolver on Fly.io, running Node, is backed by [persistent VMs](https://fly.io/blog/docker-without-docker/) and is hence longer-lived,
like traditional "serverfull" environments.

For Deno Deploy, the code-base is bundled up in a single javascript file with `deno bundle` and then handed off
to Deno.com.

Cloudflare Workers build-time and runtime configurations are defined in [`wrangler.toml`](wrangler.toml).
[Webpack5 bundles the files](webpack.config.cjs) in an ESM module which is then uploaded to Cloudflare by _Wrangler_.

Fastly Compute@Edge build-time and runtime configurations are defined in [`fastly.toml`](fastly.toml).
[Webpack5 bundles the files](webpack.fastly.cjs) in an ESM module which is then compiled to WASM by `npx js-compute-runtime`
and subsequently packaged and published to Fastly Compute@Edge with the _Fastly CLI_.

For Fly.io, which runs Node, the runtime directives are defined in [`fly.toml`](fly.toml) (used by `dev` and `live` deployment-types),
while deploy directives are in [`node.Dockerfile`](node.Dockerfile). [`flyctl`](https://fly.io/docs/flyctl) accordingly sets
up `serverless-dns` on Fly.io's infrastructure.

```bash
# build and deploy for cloudflare workers.dev
npm run build
# usually, env-name is prod
npx wrangler publish [-e <env-name>]

# bundle, build, and deploy for fastly compute@edge
# developer.fastly.com/reference/cli/compute/publish
fastly compute publish

# build and deploy to fly.io
npm run build:fly
flyctl deploy --dockerfile node.Dockerfile --config <fly.toml> [-a <app-name>] [--image-label <some-uniq-label>]
```

For deploys offloading TLS termination to Fly.io (`B1` deployment-type), the runtime directives are instead defined in
[`fly.tls.toml`](fly.tls.toml), which sets up HTTP2 Cleartext and HTTP/1.1 on port `443`, and DNS over TCP on port `853`.

Ref: _[github/workflows](.github/workflows)_.

### Blocklists

190+ blocklists are compressed in a _Succinct Radix Trie_ ([based on Steve Hanov's impl](https://stevehanov.ca/blog/?id=120)) with modifications
to speed up string search ([`lookup`](https://github.com/serverless-dns/trie/blob/965007a5c/src/ftrie.js#L378-L484)) at the expense of "succintness". The blocklists are versioned
with unix timestamp (defined in `src/basicconfig.json` downloaded by [`pre.sh`](src/build/pre.sh)), which is generated once every week, but we'd like to generate 'em daily / hourly,
if possible [see](https://github.com/serverless-dns/blocklists/issues/19)), and hosted on Cloudflare R2 (env var: `CF_BLOCKLIST_URL`).

`serverless-dns` downloads [3 blocklist files](https://github.com/serverless-dns/serverless-dns/blob/15f62846/src/core/node/blocklists.js#L14-L16)
required to setup the radix-trie during runtime bring-up or, downloads them [lazily](https://github.com/serverless-dns/serverless-dns/blob/02f9e5bf/src/plugins/dns-op/resolver.js#L167),
when serving a DNS request.

`serverless-dns` compiles around ~13M entries (as of Jan 2023) from around 190+ blocklists. These are defined in the [serverless-dns/blocklists](https://github.com/serverless-dns/blocklists) repository.






Open Source
The free and open source RethinkDNS Resolver is serverless and supports protocols: DNS over HTTPS (DoH) & DNS over TLS (DoT). It can be configured with custom blocklists. And can be hosted on cloudflare, fly.io or deno-deploy. Source code is made available at github.com/serverless-dns/serverless-dns.

Hosting your own DNS Resolver#
This serverless DNS can be hosted to three platforms: Cloudflare, Deno-Deploy and Fly.io. Easiest way would be to use Cloudflare. The below table summarizes the platforms available.

Platform	Difficulty	Resolver Protocol	Instructions
⛅ Cloudflare	Easy	DoH	Read Instructions
🦕 Deno Deploy	Moderate	DoH	Read Instructions
🪂 Fly.io	Hard	DoH and DoT	Read Instructions
Using Cloudflare#
Rethink serverless can be hosted to cloudflare. User will be liable for cloudflare billing. Click the button below to deploy.

Deploy to Cloudflare Workers

Configure
Once the hosting is successful, lets consider rethink serverless dns is hosted to example.com.
To configure your dns level blocking visit to example.com/configure which will take to configuration page, which currently contains 191 blocklists with ~13.5 Million blockable domains in category like notracking, dating, gambling, privacy, porn, cryptojacking, security ...
Navigate through and select your blocklists.
Once selected you can find your domain name example.com followed by configuration token on screen like this https://example.com/1:AIAA7g== copy it and add to your dns DOH client.
Now your own trusted dns resolver with custom blocking is up and running.
Change Resolver
By default dns request are resolved by cloudflare cloudflare-dns.com.
To change resolver login to your cloudflare dash board
click on worker
click on serverless-dns worker
click on Settings tab
under Environment Variables click on Edit variables
if your new DOH resolver url is example.dns.resolver.com/dns-query/resolve
change below variables and click on save button CF_DNS_RESOLVER_URL = example.dns.resolver.com/dns-query/resolve
alternatively, you can modify your CF_DNS_RESOLVER_URL in your forked serverless-dns repo. Head to src/core/env.js and modify either your CF_DNS_RESOLVER_URL or CF_DNS_RESOLVER_URL_2 DNS server
Using Deno-Deploy#
This project can be hosted on deno.com/deploy and supports DoH only. User will be liable for deno.com billing.

Fork the serverless-dns repository (requires a GitHub account).
In the repository you just forked, click on the Actions tab and Confirm that you want to use Actions, if prompted.
Now, navigate to deno.com/deploy and Sign Up for an account.
Create a new project via the deno dashboard by just clicking on the New Project button.
Select your forked repository from the list of repositories that appear. If your GitHub username does not show up, click on + Add GitHub Account and install the Deno Deploy GitHub App with the necessary permissions.
Once you select your forked repository, input boxes for project configuration show up. You need to do two things here; enter deno task prepare in the Build Step, and select src/server-deno.ts in Entrypoint field.
Click on the Deploy Project button.
Once your project is successfully deployed, you will see "Success!" page with a URL to your deployment that looks like https://<project-name>.deno.dev. If you found any issues, reach out to us over on GitHub.
Optionally, if you need to configure environment variables, you can do so in the project settings page that you can access either by clicking on the Add environment variables link shown in the "Success!" page, or by navigating to https://dash.deno.com/projects/<project-name>/settings#environment-variables.
Using Fly.io#
This project can be hosted on fly.io, and can support both DoT and DoH protocols. User will be liable for fly.io billing.

Install flyctl on your device (ref).
Signup and/or login to fly.io (ref).
Create an empty directory anywhere on your PC. Open you terminal or powershell and navigate to this directory.
Launch a fly app
flyctl launch
Choose a unique name here or let it auto-generate.
Choose a location (the closer to your users, the better).
Note down the name of the app and you may delete this directory along with the generated fly.toml.
Now, you would need a SSL or TLS certificate for your domain name. Both getting a domain name and CA certificate generation are beyond the scope of this README. Let's Encrypt and ZeroSSL vend public certs for free, valid for a maximum of 90 days.
Once you have your CA certificate and key files, you need to encode them as base64 with no wrapping. How this can be done in bash terminal is shown below.
# Locate your CA certificate & key files
CRT="path/to/full-chain-certificate.pem"
KEY="path/to/key.pem"
# Encode them in base64 with no wrappings and store them in variables
B64NOWRAP_KEY="$(base64 -w0 "$KEY")"
B64NOWRAP_CRT="$(base64 -w0 "$CRT")"
As described in .env.example file, this base64 encoded certificate-key pair need to set as a single environment variable called TLS_. Within this variable, the certificate and key encodings needs to be separated by a newline (\n) and described by CRT= and KEY=. On a bash terminal this can be done by following steps continued by by above.
# This creates a single file called "FLY_TLS" in the current directory
echo "KEY=$B64NOWRAP_KEY" > FLY_TLS
echo "CRT=$B64NOWRAP_CRT" >> FLY_TLS
# And now, this "FLY_TLS" file contains both certificate and key encoded and
# as required
Upload this to fly secrets like so in terminal or powershell:
fly secrets set TLS_CERTKEY=- < FLY_TLS -a <app-id>
where <app-id> is the name of the fly app you had launched in step 4.
Other essential environment variables are already present in fly.toml file of this repository, but you may read .env.example for it's use case and configuration.
Fork the serverless-dns repository (you will need a GitHub account).
In your fork, click on the Actions tab and Confirm that you want to use Actions, if asked.
Similarly, click on Settings tab and select Secrets on the left pane. Add a new GitHub secret called FLY_APP_NAME and set it's value as the name of the fly app you had launched in step 4. And add another secret called FLY_API_TOKEN and set's value as what you get from running flyctl auth token in terminal or powershell.
Head back to Actions tab and click on Fly on the left pane. Click on the Run workflow dropdown on the right side, and run the workflow using the Run workflow button.
Once this action workflow finishes, open the terminal or powershell again and type in:
flyctl ips list -a <app-id>
where <app-id> is the name of the fly app you had launched in step 4.
Here, you can get the IP address of the application, update the DNS records of your domain name you had used in step 5.
Done. Your application should be available on the said domain name in a few minutes. To configure, for example, the upstream resolver, you can edit the environment variables in the fly.toml file of your fork and re-run the Action workflow.
Edit this page
