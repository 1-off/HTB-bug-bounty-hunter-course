

## Domains and Subdomains
Bug bounty programs will often set the scope as something such as *.inlanefreight.com, meaning that all subdomains of inlanefreight.com, in this example, are in-scope (i.e., acme.inlanefreight.com, admin.inlanefreight.com, and so forth and so on). We may also discover subdomains of subdomains. For example, let's assume we discover something along the lines of admin.inlanefreight.com. We could then run further subdomain enumeration against this subdomain and perhaps find dev.admin.inlanefreight.com as a very enticing target. 
There are many ways to find subdomains (both passively and actively) which we will cover later in this module.

## ip ranges
Unless we are constrained to a very specific scope, we want to find out as much about our target as possible. Finding additional IP ranges owned by our target may lead to discovering other domains and subdomains and open up our possible attack surface even wider.

## Infrastructure

## Virtual Hosts
Lastly, we want to enumerate virtual hosts (vhosts), which are similar to subdomains but indicate that an organization is hosting multiple applications on the same web server. We will cover vhost enumeration later in the module as well.


## Passive gathering
We do not interact directly with the target at this stage. Instead, we collect publicly available information using search engines, whois, certificate information, etc. The goal is to obtain as much information as possible to use as inputs to the active information gathering phase.

## Active information gathering
We directly interact with the target at this stage. Before performing active information gathering, we need to ensure we have the required authorization to test. Otherwise, we will likely be engaging in illegal activities. Some of the techniques used in the active information gathering stage include port scanning, DNS enumeration, directory brute-forcing, virtual host enumeration, and web application crawling/spidering.

## whois
```bash
whois <domain> | grep -e 'regex here'
```

## DNS lookup
```bash
export TARGET=www.facebook.com
dig facebook.com @1.1.1.1
dig a www.facebook.com @1.1.1.1
dig -x 31.13.92.36 @1.1.1.1
dig any google.com @8.8.8.8
nslookup -query=A $TARGET
nslookup -query=PTR 31.13.92.36
nslookup -query=ANY $TARGET
```

## Passive Subdomain Enumeration
- https://censys.io
- https://crt.sh

```bash
export TARGET="facebook.com"
curl -s "https://crt.sh/?q=${TARGET}&output=json" | jq -r '.[] | "\(.name_value)\n\(.common_name)"' | sort -u > "${TARGET}_crt.sh.txt"

curl -s 	Issue the request with minimal output.
https://crt.sh/?q=<DOMAIN>&output=json 	Ask for the json output.
jq -r '.[]' "\(.name_value)\n\(.common_name)"' 	Process the json output and print certificate's name vale and common name one per line.
sort -u 	Sort alphabetically the output provided and removes duplicates.

export TARGET="facebook.com"
export PORT="443"
openssl s_client -ign_eof 2>/dev/null <<<$'HEAD / HTTP/1.0\r\n\r' -connect "${TARGET}:${PORT}" | openssl x509 -noout -text -in - | grep 'DNS' | sed -e 's|DNS:|\n|g' -e 's|^\*.*||g' | tr -d ',' | sort -u
```


