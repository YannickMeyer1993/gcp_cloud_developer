`sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
`

This command (above) to configure the IP tables redirects requests on Port 80 to Port 8080 - the Java Web application listens on Port 8080.
