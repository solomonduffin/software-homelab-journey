So my goal with this app is to make a finance tracker to aggregate all of my personal finance data. It'll be good practice too. I want a self-hosted Docker application that connects into all of my different finance apps and displays the information I want displayed. We'll start simple but maybe make it more complex as I need. Ideally I can make it public and generalized enough to work with any different finance accounts, but we'll see. 

I'm using SimpleFIN to grab all of the data. SimpleFIN has a pretty simple protocol to grab read-only JSON data from financial institutions. https://www.simplefin.org/protocol.html

For the backend, I'll use Golang to fetch the info, and to make restful endpoints for the app using go-chi. If I was just going for speed I'd probably use Python FastAPI, or Node.js as alternative options, but I want to practice using Go. One idea for more practice is to build out a few backends, and allow my frontend to switch between them. This would ensure my front and backends are properly decoupled and continue to work when swapping between them. Could also be cool to see how I'd build them out on each, especially for a simple app.

I'll have a cronjob 'Goroutine' thread for fetching data once a day and storing it in the DB.

I'll use SQLite for the DB because it doesn't need to be robust at all. Small amounts of data and not separated over the network.

And for the frontend I think I'll use React. We used Go and React Native for our chess capstone project, so I want to nail these tools in while they're fresh.

I'll make all of this containerized where it makes sense to practice using Docker. Also best for self-hosted applications for sure.

## Clearing up some concepts:

I had some confusion around Nginx (and still do a bit), so here's the gist of what it is and why it's necessary. It wears a few hats.
1. Web Server Hat - When we go to a specific URL, say finance.local, or google.com, something has to serve the correct files to the browser. This is a web server.
2. Reverse Proxy Hat - The Reverse Proxy is the traffic cop sending requests where they need to go. If my frontend wants to grab specific information from my backend, say the finance JSON, we use the REST API endpoints to do so. So maybe we call finance.local/data. That's where our endpoint config comes in, where Nginx will see that we're calling GET /data and will direct traffic specifically to the Go container on port :8080 or whatever it needs to be.

Nginx is also a secure point of contact to the outside world. According to Gemini: "It is designed to get hammered by the raw internet, drop malicious packets, and manage your SSL/TLS certificates (HTTPS). You want Nginx standing on port 443 handling the encryption, and then passing plain, safe HTTP traffic back to your Go app on port 8080."
