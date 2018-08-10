# Scenario: 
I am a warehousing company. On the first and third Friday of every month, our sales spike due to members’ paydays. During these times, people checking our produce inventory within the stores (and online) goes up significantly so we want to make sure that we are able to accurately display inventory so we are not selling produce that we do not have enough of. Anecdotal evidence from cahiers and customers claim inventory lookups are slow during busy peak times. The current strategy is to add more resources to the database cluster but I want to assure the business that we are ready to scale at a reasonable cost.

 

# Assumptions:
All warehouse stores have network connectivity to database resources, cloud, and local data centers
The business will want to provide fault tolorance leveraging on-premise servers and cloud resources
 Provide 99.99% uptime with <5 second response time for a reasonable cost
 Tools to Use:

Vagrant - Using standardized OS images to test continuous build's will free up resources (developers and testers) - no need for dedicated environments. Vagrant image/build will include local software load balancer, database, inventory app, and API. 
 Consol - Manage both on-prem and cloud infrastructure - pushing continous builds with a minimum of impact
 Docker/Kubernetes - Containerization will reduce vendor lock-in while allowing for on-prem hardware scaling and cloud resource. Leveraging application based caching solutioins (redis for example) while running in multiple regiions and local data centers, will ensure rapid response as well as the 99.99% uptime.
 AWS/Azure/GCP - Containerized approach will allow for relevant region specific resources expand and contract as needed while keeping the costs at a minimum
 Cloud ready datastore - Leveraging key value index stores (example - Cassandra, Redis, Hbase, etc) will be leveraged for rapid updates with quick response times. BI functionality to be provided with ODS Snowflake DB
Should I look at any kind of API call? Yes, the workflow I envision is a webapp running onsite at the warehouses as well as customer mobile devices. So the workflow would be - Mobile device webapp-->API-->Datastore. Running containerized API apps, you will be able to run in the cloud or on-prem without penalty. 

I threw it out and the “boss to be” was very interested in this and even brought up using a LAMP stack (which I need to read up on).- LAMP stands for Linux, Apache (webserver), MySQL, and PHP. This has been proven to not scale and is overly complex. I abandoned the LAMP stack approach years ago in favor of a serverless approach - AWS S3 hosting static content and JS, with CloudFront CDN - wicked fast without any DB failures and super low cost. PHP is great, but very hard to scale. You have to put memcached or caching in front of it.

I also read up on Ansible but not sure if needed/desired - Ansible is used for configuration management, great product but is in the same space with CHEF, Puppet, etc. The need for configuration management is decreased when you move towards containerization - since you are running just enough OS for the app to run, why manage an entire OS with CHEF, Puppet, Ansible, etc. Configuration management is better than nothing - but I think with our approach we can knock it out with Vagrant for local developement, Consol for infrastructure managment, caching, Kubernetes to scale, and a wicked fast eventually consistent, distributed, datastore to bring the data closer to the users.
