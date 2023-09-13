# OCI Associate

1. ## OCI Fundamentals

    Questions:

    1. Which capability can be used to protect against failures within an OCI availability domain?
        - Load Balancer
        - Regions
        - Fault Domain [Right]
        - Compartments
        > Explanation: Fault domains provide a capability to protect your applications and instances from unexpected hardware failures or network outages within an availability domain. They provide anti-affinity: Each fault domain runs on its own set of physical hardware, so a failure that impacts one fault domain does not affect instances in other fault domains.

    2. You have subscribed to an OCI region that has one availability domain. You want to deploy a highly available application with two servers and a 2-node database. How would you place the components to maintain the high availability of the application?
        - Place one server and a DB node in one fault domain, and the second server and DB node in another fault domain. (*)
        - Place the servers in one fault domain and the database nodes in another fault domain.
        - Place all the components in the same fault domain.
        - High availability is not possible as there is only one availability domain in the region.
        > In this scenario, distributing the servers and database nodes across different fault domains within the same availability domain would provide protection against the failure of a single fault domain. If one fault domain experiences a failure, the other would remain unaffected, ensuring the high availability of the application.

    3. Which Oracle Cloud Infrastructure service is NOT intended for a multicloud solution?
        - Oracle Database Service for Azure
        - Oracle Interconnect for Azure
        - Oracle MySQL Heatwave on AWS
        - Oracle Roving Edge Infrastructure (*)
        >Incorrect. Oracle Roving Edge Infrastructure is a service that provides a portable, ruggedized device running a subset of OCI services, designed for deployment in the field outside of a traditional data center. It is not a service specifically designed for multicloud deployment. On the other hand, services like Oracle Database Service for Azure and Oracle Interconnect for Azure are designed to allow Oracle Cloud Infrastructure to interoperate with Azure, indicating a multicloud approach. Oracle MySQL HeatWave is an analytics service for MySQL Database service that runs on AWS but the account management and billing and metering are done through OCI.

    4. Which statement about OCI is NOT true?
        - An availability domain is one or more data centers located within a region.
        - An OCI region is a localized geographic area.
        - A single fault domain can be associated with multiple availability domains within a region. (*)
        - Availability domains do not share infrastructure, such as power, cooling, or network, within a region.
        >Correct. A fault domain is a subdivision of an availability domain. Each availability domain contains three fault domains. Fault domains let you distribute your instances so that they are not on the same physical hardware within a single availability domain. A fault domain cannot be associated with multiple availability domains.

    5. Which statement about regions and availability domains is true?
        - All OCI regions have a single availability domain.
        - Fault domains provide protection against failures across regions.
        - All OCI regions have three availability domains.
        - An OCI region has one or more availability domains. (*)
        >Correct. An OCI region is composed of one or more isolated, interconnected availability domains. Each availability domain is a separate physical location within a region. The number of availability domains per region may vary; some OCI regions have three availability domains, while some others have a single availability domain.

