# Well architected Framework

5 pillars :
- Operational excellence
   - Operations as code
   - Annotate documentation
   - Make frequent small and reversible change
   - Refine operations procedures frequently
   - Learn from all operation failure
- Reliability
   - Test recovery procedure
   - Automaticly recover from failure
   - Scale horizontaly
   - Stop guessing capacity
   - Automate change
- Security
   - Strong identity fondation
   - Tracability
   - Apply security at every level
   - Automate security
   - Protect data in transit and at rest
   - Prepare for security events
- Performance efficiency
   - Democratize advanced technologie
   - Go global in minute
   - Use serverless architecture
   - Experiment more often
- Cost optimization
   - Adopt consumption model
   - Mesure overall efficiency
   - Stop spending money on datacenter
   - Analyze and attribute expenditures
   - Use managed services






# AWS architecture
## Regions
Each region is independent. The price of services may change regarding the region you use

## Availibility zones
Grouped in a region (18 active regions now) and separated from few miles

## Edge locations
Are not affiliated to a region. They are AWS Datacenter PoP that operates only 2 services:
- Route 53
- CloudFront

## Regional Edge Cache
A frequently accessed data is cached in an Edge location, then when this data is supposed to be removed from the Edge cache location (because lessly used), it is cached in the Regional Edge Location that is present in a region




# Services
## Route 53
DNS

## CloudFront
Content Delivery CDN.

It contains few services :
- AWS Shield : Anti-DDoS
- AWS WAF : Web Application Firewall
- Lambda@Edge
- S3 Transfert  
- API Gateway 

