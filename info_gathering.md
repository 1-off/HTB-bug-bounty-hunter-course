
# Domain and subdomains
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
- https://github.com/laramies/theHarvester

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

### harvester
```bash
cat sources.txt

baidu
bufferoverun
crtsh
hackertarget
otx
projecdiscovery
rapiddns
sublist3r
threatcrowd
trello
urlscan
vhost
virustotal
zoomeye
```
```bash
export TARGET="facebook.com"
cat sources.txt | while read source; do theHarvester -d "${TARGET}" -b $source -f "${source}_${TARGET}";done
```
Extract all subdomains
```bash
cat *.json | jq -r '.hosts[]' 2>/dev/null | cut -d':' -f 1 | sort -u > "${TARGET}_theHarvester.txt"
cat facebook.com_*.txt | sort -u > facebook.com_subdomains_passive.txt
cat facebook.com_subdomains_passive.txt | wc -l
```
---------------------------------------------------------------
# Passive Infrastructure Identification
Visiting ```https://sitereport.netcraft.com/?url=```
Some interesting details we can observe from the report are:
- Background 	General information about the domain, including the date it was first seen by Netcraft crawlers.
- Network 	Information about the netblock owner, hosting company, nameservers, etc.
- Hosting history 	Latest IPs used, webserver, and target OS. Sometimes we can spot the actual IP address from the webserver before it was placed behind a load balancer, web application firewall, or IDS, allowing us to connect directly to it if the configuration allows it. 

- checking old versions: https://web.archive.org/  It can very likely lead to us discovering forgotten assets, pages, etc., which can lead to discovering a flaw.
```bash
go get github.com/tomnomnom/waybackurls
waybackurls -dates https://facebook.com > waybackurls.txt
cat waybackurls.txt
```

--------------------------------------------------------------
# Active Infrastructure Identification
## Web servers
### HTTP Header
X-Powered-By header: This header can tell us what the web app is using. We can see values like PHP, ASP.NET, JSP, etc.
Cookies: Cookies are another attractive value to look at as each technology by default has its cookies. Some of the default cookie values are:
```bash
curl -I 'http://${TARGET}'
HTTP/1.1 200 OK
Date: Thu, 23 Sep 2021 15:10:42 GMT
Server: Apache/2.4.25 (Debian)
X-Powered-By: PHP/7.3.5
Link: <http://192.168.10.10/wp-json/>; rel="https://api.w.org/"
Content-Type: text/html; charset=UTF-8


HTTP/1.1 200 OK
Host: randomtarget.com
Date: Thu, 23 Sep 2021 15:12:21 GMT
Connection: close
X-Powered-By: PHP/7.4.21
Set-Cookie: PHPSESSID=gt02b1pqla35cvmmb2bcli96ml; path=/ 
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-type: text/html; charset=UTF-8
```
#### https://github.com/urbanadventurer/WhatWeb
Recognizes web technologies, including content management systems (CMS), blogging platforms, statistic/analytics packages, JavaScript libraries, web servers, and embedded devices.
```bash
whatweb -a3 https://www.facebook.com -v
```

#### https://www.wappalyzer.com/
Find out the technology stack of any website. Create lists of websites that use certain technologies, with company and contact details. Use our tools for lead generation, market analysis and competitor research.

#### WafW00f 
it is a web application firewall (WAF) fingerprinting tool that sends requests and analyses responses to determine if a security solution is in place. We can install it with the following command
```bash
sudo apt install wafw00f -y
wafw00f -v https://www.tesla.com
```
#### Aquatone
It is a tool for automatic and visual inspection of websites across many hosts and is convenient for quickly gaining an overview of HTTP-based attack surfaces by scanning a list of configurable ports, visiting the website with a headless Chrome browser, and taking and screenshot.
```bash
sudo apt install golang chromium-driver
go get github.com/michenriksen/aquatone
export PATH="$PATH":"$HOME/go/bin"

cat facebook_aquatone.txt | aquatone -out ./aquatone -screenshot-timeout 1000
```
