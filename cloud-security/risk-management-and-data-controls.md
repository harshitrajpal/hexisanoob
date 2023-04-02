# Risk Management & Data Controls

IaaS (amazon EC2, Azure) storage types:

1. Raw storage - Direct access to underlying storage (not available in cloud)
2. Object storage - File storage in the cloud. Eg: Amazon S3, Dropbox
3. Volume Storage - Virtual hard drive which uses data dispersion. Eg: Amazon Block-Level Storage (EBS).
4. Content Delivery Network - Uses object storage to store content across multiple geographically dispersed caches. Eg: Amazon Cloudfront.

Functions: Access (Create, Copy, Transfer, read), Process(update), Store(hold the data)

Data actors: Who or what is performing the given function.

Controls: Restrict which functions an actor is allowed to perform



"Who can access the data, where is the actor and where is the data?"



### Data Security Epics

1. Access Controls
2. Encryption
3. Data Activity Monitoring
4. File Activity Monitoring
5. URL filtering
6. Data Loss Prevention
7. Cloud Access Service Broker
8. Digital Rights Management
9. Privacy Preserving Storage

**IaaS volume Storage Encryption**

1. Volume Encryption - addresses: Snapshots, Physical Loss of Drive, Cloud Provider access
2. Volume Encryption - techniques: \\\\
   1. Managed - keep the key in the volume which is encrypted. Key protected with passphrase. Trucrypt for example.\\
   2. External Managed - Key management system which is separate from the volume

**IaaS Object Storage Encryption**

Object storage encryption can act as a Virtual Private Storage. Data stored in cloud but only key holder can read the data.

## Standards

### NIST 500-292 - Shows the shift in responsibility and controls between the various cloud computing models.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

### CSA Cloud Security Guidance

Cloud Security Alliance (CSA ) is a non-for-profit organization formed to promote best practices in providing security assurance with cloud computing environments • Provide education on the uses of cloud technologies with focus on security • 14 Domains covering architecture, governance and cloud operations



### CSA Governance Domains

• Governance and Enterprise Risk Management • Legal Issues • Compliance and Audit • Information Management and Data Security • Portability and Interoperability



### CSA Operational Domains

Traditional security, business continuity and disaster recovery • Data center operations • Incident Response • Application Security • Encryption and Key Management • Identity and Access Management • Virtualization • Security as a Service



Some important CSA Domains

1. **Governance and Enterprise Risk Management** - Reinvest the cost savings obtained by cloud computing services into increased scrutiny of the security capabilities of the provider, application of security controls, and ongoing detailed assessments and audits to ensure requirements are continuously met. • Metrics and standards for measuring performance and effectiveness of information security management should be established prior to moving into the cloud. If the services provided in the cloud are essential to corporate operations a risk management approach should include identification and valuation of assets
2. **Legal Issues** - • E-Discovery • Preservation • Forensics • Integrity of the Data • Admissibility
3. **Compliance and Audit Management** - Understand the contractual responsibilities of each party. • Prefer auditors that are "cloud aware"
4. **Interoperability and Portability** - \
   Whenever possible, use virtualization to remove many hardware level concerns \
   • To maintain interoperability the Network physical hardware and network & security abstraction should be in virtual domain. \
   • Using open virtualization formats such as OVF to help ensure interoperability. \
   • Store unstructured data in an established portable format. \
   • Use SAML or WS-Security for authentication so the controls can be interoperable with other standards-based systems. \
   • Encrypting data before it is placed into the cloud will ensure that it cannot be accessed inappropriately within cloud environments. \
   • Encryption keys should be escrowed locally, and when possible maintained locally
5. **Traditional Security, Business Continuity and Disaster Recovery** -\
   • Cloud providers should consider adopting as a security baseline the most stringent requirements of any customer. \
   • Providers should have robust separation of job duties, perform background checks, require and enforce nondisclosure agreements for employees, and restrict employee knowledge of customers to a least privilege, need to know basis. \
   • Transparency regarding the security posture of the CSP should be required. \
   • To enhance effectiveness of the onsite assessment, the visit to the CSP facility or data center should be carried out unannounced \
   • Consumers should check to see if the CSP deploys competent security personnel for its physical security function. \
   • The customer should review the contract of third party commitments to maintain continuity of the provisioned service. \
   • The customer should ensure that he/she receives confirmation of any BCP/DR tests undertaken by the CSP \
   • Cloud customers should not depend on a singular provider of services and should have a disaster recovery plan in place that facilitates migration or failover should a supplier fail.
6. Data Center Operations -\
   • Organizations building cloud data centers should incorporate management processes, practices, and software to understand and react to technology running inside the data center. \
   • Data center locations are important: Local regulation, latency. \
   • Organizations buying cloud services must clearly understand and document which parties are responsible for meeting compliance requirements \
   • Insure data center is being designed to be agile and in line with Next Generation Data Center requirements.
7. Application Security -\
   • A detailed assessment of the attack vectors and risks in the cloud environment are understood, and the mitigations strategies are integrated into the requirements. \
   • An impact assessment for all risks and attack vectors is undertaken, and documented, together with the potential for loss or damage from each scenario. \
   • Risk analysis of the applications for security and privacy \
   • Attack vectors and impact analysis specific to cloud architectures should be catalogued and maintained.
8. Encryption and Key Management -\
   • Use best practice key management practices when using any form of encryption/ decryption product. \
   • Use off-the-shelf technology where possible to get the best practices from a credible source. \
   • Use standard algorithms. Do not invent/use proprietary scrambling techniques. \
   • Use object security (unless not advised by provider). You should still use basic object security (SQL grant and revoke statements) to prevent access to even the encrypted data.
9. Identity Entitlement and Access Management -\
   • Implementers should understand what trust relationship and transitive trust exist \
   • Implementers should, where possible, use federation based on open standards such as SAML and OAuth. \
   • As a rule, the cloud service or application itself should avoid being the master source for Identity. \
   • The cloud service or application itself should only be the master source for Attributes it directly controls. \
   • Cloud services and applications should support the capability to consume authentication from authoritative sources using SAML. \
   • Implementers should follow the rule of least privilege when provisioning an account. \
   • Implementers should ensure that the applicable logs from the entitlement rules / authorization process are capable of being made available. \
   • The implementer should consider the following technologies to minimize exposure of PII or SPI: Encryption, Tokenization, Homomorphic Encryption
10. Virtualization - \
    Customers should identify which types of virtualization the cloud provider uses, if any \
    • Implementers should consider a zoned approach with production environments separate from test/development and highly sensitive data/workloads. \
    • Implementers should consider performance when testing and installing virtual machine security tools \
    • Implementers should encrypt virtual machine images when not in use.
11. Security as a Service - \
    • Implementers should ensure secure communication channels between tenant and consumer. \
    • Providers should supply secured logging of internal operations for service level agreement compliance. \
    • All parties should enable Continuous Monitoring of all interfaces through standardized security interfaces

## AWS Compliance Service

SecurityHub Security Standard.

• Set of controls that detect when your deployed accounts and resources deviate from security best practices.The standard allows you to continuously evaluate all of your AWS accounts and workloads to quickly identify areas of deviation from best practices. It provides actionable and prescriptive guidance on how to improve and maintain your organization’s security posture. • Each control in AWS Security Hub is assigned a category.&#x20;

The category for a control reflects the security function that the control applies to.&#x20;

• The category value contains the category, the subcategory within the category, and, optionally, a classifier within the subcategory.&#x20;

For example:&#x20;

• Detect > Detection services > Application monitoring&#x20;

• Identify > Inventory&#x20;

• Protect > Data protection > Encryption of data in transit&#x20;



PCI DSS Standard for those who deal with credit card data&#x20;

CIS AWS Foundations Benchmark standard



Other good frameworks:

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

## Risk Management in Cloud Environments

• Identify the Asset for cloud deployment - Data, Applications/functions/process

• Evaluate the asset – how sensitive the data, importance of the applications, process to business

• Damage to business if: • Asset became publicly available • Employee of cloud provided had access • App or process manipulated by an outsider • Data manipulated • Asset data not available

• Map the asset to potential cloud deployment models

• Evaluate potential cloud service models and providers

<figure><img src="../.gitbook/assets/image (159) (1).png" alt=""><figcaption></figcaption></figure>

## Risk Assessment

• Assessment: measures the impact of an event, and the probability of an event (threat agent exploiting a vulnerability) \


• Quantitative (objective) and Qualitative (subjective) approaches both used. \


• Quantitative approach: • Compute expected monetary value (impact) of loss for all “events” • Compute the probability of each type of expected loss \


• Qualitative approach: use Low, Medium, High; ratings; other categorical scales



## Risk Management

• Accept the risk – Risk is low but costly to mitigate - worth accepting. Monitor. \
• Transfer the risk – Transfer to somebody else via insurance, warnings etc. \
• Remove the risk - Remove the system component or feature associated with the risk if the feature is not worth the risk. \
• Mitigate the risk - Reduce the risk with countermeasures. \
• The understanding of risks leads to policies, specifications and requirements. \
• Appropriate security mechanisms are then developed and implemented



## Quantitative - Security Cost Risk Assessment

• Exposure Factor (EF) - Percentage of asset loss caused by identified threat \
• Single Loss Expectancy (SLE) - Asset Value X Exposure Factor \
• Annualized Rate of Occurrence (ARO) - Estimated frequency a threat will occur within a year \
• Annualized Loss Expectancy (ALE) - Single Loss Expectancy x Annualized Rate of Occurrence



<figure><img src="../.gitbook/assets/image (160).png" alt=""><figcaption></figcaption></figure>

### Quantitative Pros and Cons

• Pros: • Objective, independent process • Credibility for audit, management (especially corporate management) • Solid basis for evaluating cost/benefit of countermeasures • Quantitative risk assessment is the basis for insurance, risk managed portfolios, etc.&#x20;

• Cons: • Not reliable for “rare” events or “unthinkable” impacts • Sometimes difficult to get solid numbers for ARO



## Qualitative&#x20;

Establish classes of loss values (“impact”), such as&#x20;

• Low, medium, high&#x20;

• Under $10K, between $10K and $1M, over $1M (used by at least one company)&#x20;

• Type of loss (e. g. compromise of credit card #, compromise of SSN, compromise of highly personal data)&#x20;

• Minor injury, significant injuries, loss of life, large scale loss of life (used by emergency response organizations to categorize non-IT events)&#x20;

• Rank ordering

Establish classes of likelihood of compromise \
• Low, medium, high likelihood \
• Decide on a risk management approach to each combination of (class of loss, likelihood of loss) \
• Focus effort on medium to high loss and/or medium to high likelihood items
