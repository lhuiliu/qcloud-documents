- Currently, available regions include Beijing, Shanghai, Guangzhou, Hong Kong, Toronto.
- Each user is able to create up to 20 scaling configurations in each region.
- Each user is able to create up to 20 scaling groups.
- A scaling group can only correspond to one scaling configuration.
- For all regions and scaling groups, each user can configure auto scaling for up to 30 CVM instances.
- Up to 100 scaling policies and 10 scheduled tasks can be created in each scaling group.
- The number of sub machines in scaling group cannot exceed the number of IPs that the VPC subnet is able to provide.
- Currently, auto scaling does not support vertical scaling (increasing/reducing CPU, memory and bandwidth of CVMs).
- Auto scaling and scaling configuration are regional concepts, which means they can only enable/terminate CVM instances in the same region.

