# Real Estate Finder Platform Network Architecture

## Assumptions
- **User Base**: The platform supports users searching, bidding, and chatting, with concurrent users ranging from 100 to 100,000 and monthly users from 100,000 to 100 million.
- **Regions and Availability Zones**: Deployed in two regions (Singapore and Mumbai), each with two availability zones for high availability and fault tolerance.
- **Third-Party Services**: Integration with external APIs (e.g., property listings, payment gateways, messaging services) requires low-latency access and secure connections.
- **Development Environment**: Developers need a staging environment mirroring production for testing and deployment.
- **Traffic Patterns**: Search and bidding are read-heavy, while chat is real-time and write-intensive.
- **Data Volume**: Property data includes listings, images, and metadata; chat data is lightweight but frequent.
- **Scalability**: The system must scale horizontally to handle peak loads.
- **Security**: HTTPS, encryption for data in transit, and role-based access control are required.
- **Budget**: Cost optimization is critical, balancing performance and expense.

## Project Details
The real estate finder platform enables users to search for properties, bid on listings, and communicate with buyers/sellers via chat. It integrates third-party services for property data across Singapore and Mumbai, ensuring high availability and low latency. The architecture leverages AWS for cloud infrastructure, deployed across two regions (Singapore: ap-southeast-1, Mumbai: ap-south-1), each with two availability zones to ensure redundancy. The system supports both production and development environments, with auto-scaling to handle varying user loads and secure integrations for third-party APIs.

## Architecture Decisions
- **Cloud Provider**: AWS is chosen for its robust multi-region support and extensive service offerings.
- **Multi-Region Deployment**: Singapore and Mumbai regions are used, with Route 53 for latency-based routing to direct users to the nearest region.
- **Availability Zones**: Two AZs per region ensure fault tolerance; resources like EC2 instances and RDS databases are distributed across AZs.
- **Microservices Architecture**: Separate services for search, bidding, chat, and third-party integrations, deployed as Docker containers on ECS Fargate for scalability.
- **Database**: Amazon Aurora (MySQL-compatible) for property and bidding data, DynamoDB for chat messages due to its low-latency, high-throughput capabilities.
- **Caching**: ElastiCache (Redis) caches frequently accessed property data to reduce database load.
- **Load Balancing**: Application Load Balancers (ALB) distribute traffic across ECS tasks in each region.
- **Content Delivery**: CloudFront CDN caches static assets (e.g., property images) to reduce latency.
- **Third-Party Integration**: API Gateway manages secure, rate-limited access to external APIs.
- **Development Environment**: A separate VPC mirrors production for development, with access restricted via IAM roles.

## Reasoning
The multi-region setup ensures low-latency access for users in Singapore and Mumbai, with Route 53 optimizing traffic routing. Using two AZs per region provides high availability, mitigating risks of zone failures. Microservices on ECS Fargate allow independent scaling of search, bidding, and chat functionalities, accommodating their distinct load patterns. Aurora handles structured data with ACID compliance for bidding, while DynamoDB supports real-time chat. CloudFront and ElastiCache reduce latency and database costs. API Gateway secures third-party integrations, preventing abuse. The development environment from Bangladesh ensures safe testing without impacting production.

## Networking Components and Use Cases
- **VPC**: Isolates resources in each region, with subnets for public (ALB, CloudFront) and private (ECS, RDS) resources.
- **Route 53**: Directs users to the nearest region based on latency.
- **CloudFront**: Serves static content (images, UI assets) globally with low latency.
- **ALB**: Balances traffic across ECS tasks in each AZ.
- **ECS Fargate**: Runs microservices, auto-scaling based on demand.
- **Aurora**: Stores property listings and bidding data, replicated across AZs.
- **DynamoDB**: Manages chat messages for real-time performance.
- **ElastiCache**: Caches search results to reduce database queries.
- **API Gateway**: Secures and throttles third-party API requests.
- **IAM**: Restricts developer access to production and staging environments.

## Network Diagram
The architecture diagram was created using Draw.io, a free online tool, and is included in the PR's README file as `rea_estate_finder.png`. It visualizes the multi-region setup, showing Route 53, CloudFront, ALBs, ECS Fargate, Aurora, DynamoDB, ElastiCache, and API Gateway across Singapore and Mumbai regions, with two AZs each.

## Cost Estimates
The following table estimates monthly costs for different user scales, based on AWS pricing (ap-southeast-1, October 2023). Costs include compute, database, caching, networking, and third-party API calls (assumed at $0.01 per 1,000 requests).

| **Users** | **Concurrent Users** | **Monthly Users** | **EC2/ECS ($)** | **Aurora ($)** | **DynamoDB ($)** | **ElastiCache ($)** | **CloudFront ($)** | **API Gateway ($)** | **Total ($)** |
|-----------|----------------------|-------------------|-----------------|----------------|------------------|---------------------|-------------------|---------------------|--------------|
| Small     | 100                  | 100,000           | 200 (t3.medium) | 100            | 50               | 50                  | 20                | 10                  | 430          |
| Medium    | 10,000               | 1,000,000         | 2,000 (m5.large)| 500            | 200              | 200                 | 100               | 50                  | 3,050        |
| Large     | 100,000              | 100,000,000       | 20,000 (c5.xlarge) | 5,000       | 2,000            | 1,000               | 1,000             | 500                 | 29,500       |

**Notes**:
- Costs assume auto-scaling ECS tasks, Aurora read replicas, and DynamoDB on-demand pricing.
- CloudFront costs include data transfer and request pricing.
- API Gateway costs scale with third-party API call volume.
- Developer environment costs are included (~10% of total).
- Costs are approximate and may vary based on usage patterns and optimizations.
