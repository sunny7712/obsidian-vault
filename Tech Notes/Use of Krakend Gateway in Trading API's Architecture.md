![[Pasted image 20250524195307.png]]

## General Request flow

- When a request is sent from the frontend, the first layer it encounters is the `Cloudflare` where CDN caching, DNS resolution, DDoS protection and basic rate limiting is enforced.
- Then, GCP's `load balancer` directs the request to various network endpoint groups (NEGs) based on request endpoint. NEGs are a collection of ip's of various backend services. 

## KrakenD API Gateway

- Before the request reaches the actual pod where the service is running, it encounters another layer which is an API Gateway like KrakenD.
- The reason why we use this is because they are highly customizable.
- You can define configurations, have custom rate limits for specific endpoints and you can also have custom authentication logic for specific services.

## Groww Base KrakenD

- In Groww, there is a base configuration which is defined for the gateway which is honoured by default.
- You can add your own config on top of this or you can also override the base configuration for your specific case or service.