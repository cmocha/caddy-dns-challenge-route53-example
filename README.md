# Docker Setup with Caddy for Automatic HTTPS using a DNS Challenge Override Domain
## Version 1.0.0
> Objective, we have a domain that we want Automatic HTTPS and use the DNS challenge type. We also manage the DNS records where there is no available Caddy DNS Provider Module or supported API. So we will use a dns_challenge_override_domain that points to a DNS zone in AWS Route 53 where in this case a provider is supported. The dns_challenge_override_domain is a DNS record that we will delegate to Route 53 and leverage the Route 53 Caddy DNS Provider Module. This will handle the certificate generation for us. We essentially will need to create a hosted zone in Route 53 as this dns_challenge_override_domain in our Caddyfile. Then for each certificate we wish to create, we need to create a CNAME record where the parent domain DNS is managed (in this case not Route 53). The CNAME would look like _acme-challenge.value of the dns_challenge_override_domain here Then we would also need to create NS records using the value of the dns_challenge_override_domain to point to the nameservers at Route 53 for our hosted zone. The Route 53 Caddy DNS Module will programatically follow the _acme-challenge CNAME to Route 53 and add the necessary challenge records to Route 53 where it will perform the neccessary checks to allow the certificates to be generated.

# To get started, setup your DNS records accordingly.

1. Setup a Hosted Zone in Route 53 for the domain you wish to act as the dns_challenge_override_domain.
> We will use route53.example.com. When created, Route 53 will auto create NS Records and a SOA record for the hosted zone.

2. Now add the NS records to where our parent domain is managed. These records point the dns_challenge_override_domain to the AWS Route 53 hosted zone.
- NS - HOST: route53.example.com ANSWER: ns-1234.awsdns-1234.org TTL: 600
- Add the rest, there is probably 3 more.

3. For each domain/subdomain you wish to secure you need to add a CNAME record. We will add these records where the DNS for the parent domain is managed.
4. If we want to secure one.example.com, two.example.com and three.example.com we will need to add the following CNAME records.
- CNAME - HOST: _acme-challenge.one.example.com ANSWER: route53.example.com TTL: 600
- CNAME - HOST: _acme-challenge.two.example.com ANSWER: route53.example.com TTL: 600
- CNAME - HOST: _acme-challenge.three.example.com ANSWER: route53.example.com TTL: 600

- Our DNS settings are set now. We can move on configuriguring our user access for Caddy to connect to AWS Route 53.

# AWS Route 53
- We will need to create a user and get the Access Key and Secret Key and put in our Caddyfile.
- Next we need the Hosted Zone id to put in our Caddyfile.
- Last we will need to create a policy that allows the AWS IAM user to access Route 53 and we will need to specifiy our Hosted Zone in this policy. We can create a group and connect it to this policy. Then we can add our IAM user to this group. Caddy will leverage this user to perform the ncessary actions to generate the certificates.
- Now Caddy should be able to communicate with Route 53.


## Build and Start the container

- Build container with no-cache flag
```sh
cd /toprojectfolder

docker compose build --no-cache
```
- Start container
```sh
docker compose up
```

- Start container in detached mode
```sh
docker compose up -d
```
> Watch the logs after a couple minutes the certificates should be generated. If needed to troubleshoot you can docker exec into the Caddy container and poke around.
```sh
docker exec -it <container name here> sh
```

## Some Caddy Info
> https://caddyserver.com/docs/getting-started

### Caddy Start, Stop, run
> https://caddyserver.com/docs/getting-started#start-stop-run

- Start
```sh 
 caddy run
```
- Stop
```sh
 caddy stop
```
- Reload
```sh 
 caddy reload
 ```

 ## Server Notes

 - ssh root@serveriphere

 - Pull files down 
 - scp user@host:/home/test1.txt 

 - Push files up
 - scp -r /path/to/local/directory user@host:/home/folder/

 ## Simple steps to add more certificates for sub domains
 1. Update the Caddyfile with the additional domain. Add the necessary DNS records, like any A record and CNAME record for the challenge.

 2. Push files to server
 ```sh
 scp -r projectfolder user@serverip:/home
 ```

 3. SSH to server. Check that Caddyfile has your changes. Confirm the container is also still running. 
```sh
cat Caddyfile

docker ps
```
 
 4. Enter the container to reload Caddy. 
 ```sh
 docker exec -it <container name> sh
 ```

 5. Go to where caddy is setup and reload.
```sh
 cd /etc/caddy

 caddy reload
 ```
 - We now should see in our logs that the new certificate is being generated.

 ### Changelog
 - Version 1.0.0 - Initial working example.