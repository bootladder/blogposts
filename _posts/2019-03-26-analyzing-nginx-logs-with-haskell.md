---
layout: post
title: Parsing and Storing Nginx Reverse Proxy Logs with Haskell
date: 2019-03-26 15:46 -0400
---
### Counting page hits and attacks.  
&nbsp;  
&nbsp;  

### First the setup, before any log processing happens
I'm running the docker container <https://github.com/jwilder/nginx-proxy>  
Starting out with this   
```
mv /path/to/access.log /path/to/access.log.0
kill -USR1 `cat /var/run/nginx.pid`
sleep 1
[ post-rotation processing of old log file ]
```
from <https://www.digitalocean.com/community/tutorials/how-to-configure-logging-and-log-rotation-in-nginx-on-an-ubuntu-vps>  
   
&nbsp;  

But, being in a container requires a couple extra steps.  
**1. /var/run/nginx.pid is inside the container**
* Also, I don't know how to run `cat /var/run/nginx.pid` in the container, while inside the host.  
Simple solution:  create a file `send_reload_logs_signal.sh` with that 1 liner, then use this to run it:  
`sudo docker exec nginxproxy_1 /util/send_reload_logs_signal.sh`  
Where /util/ is a volume mapped dir, or you could copy it into image with the Dockerfile.
  
**2. Note, need to touch access.log after moving it, so it exists before sending the signal**  
  
&nbsp;  
&nbsp;  
### Next, it's time for the poor man's database
Instead of rotating logs eg. `log.0 , log.1 , log.2` , I'll just stick the date at the end of the file and do it once a day with a script.  
```
#!/bin/bash
mv ../logs/access.log ../logs/access.log.$(date +%Y-%m-%d-%s)
touch ../logs/access.log
docker exec nginxproxy_1 /util/send_reload_logs_signal.sh
```
&nbsp;  
### Great!  Now I have a set of files in a directory, and I can do Haskell stuff!  
&nbsp;  
The logfile format as defined in the `nginx-proxy` looks like this:  
&nbsp;  
`mydomain.com 12.34.56.789 - - [27/Mar/2019:00:52:49 +0000] "GET /Site-Map HTTP/1.1" 301 0 "http://mydomain.com" "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2272.101 Safari/537.36"`  
&nbsp;  
```
nginxParser :: Parser NginxLog
nginxParser = do
  hostname <- pHostname <* whiteSpace
  ipAddr   <- pIpAddr   <* whiteSpace
  pTwoDashes            <* whiteSpace
  date     <- pDate     <* whiteSpace
  char '\"'
  method   <- pMethod   <* whiteSpace
  endpoint <- pEndpoint <* whiteSpace
  protocol <- pProtocol <* whiteSpace
  status   <- pStatus   <* whiteSpace
  return $ NginxLog hostname ipAddr date method endpoint protocol status

```
&nbsp;  
Here's a gist for getting the hostname.  Good example!  
<https://gist.github.com/peat/2212696>
&nbsp;  
```
pHostname :: Parser String
pHostname = do
  segments <- (sepBy1 pSegment pSeparator)
  return $ intercalate "." segments
  where
    pSegment   = do segment <- many1 (alphaNum <|> char '-')
                    if last segment == '-'
                      then fail "domain segment must not end with hyphen"
                      else return $ segment
    pSeparator = char '.'
```
&nbsp;  
### But hmm, turns out to be not very interesting yet...
&nbsp;  
Parsing the log files into a record was a straight forward and good exercise but then when doing any processing or analysis I'm getting this obvious thought:  **Duh, use SQL!**  
&nbsp;  
### So the poor man's database now becomes SQLite
  
**And yet again, I forgot I've already done this before!**  
**I forgot there's an obvious layered architecture for logging**  
&nbsp;  
So let's get this out of the way and finalize the data collection.
