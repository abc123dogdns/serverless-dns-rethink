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
