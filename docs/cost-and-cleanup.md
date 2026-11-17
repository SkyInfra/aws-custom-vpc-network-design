# Cost Awareness and Cleanup

## Cost-conscious decisions

This exercise was designed to avoid optional infrastructure charges where possible.

- Both EC2 instances used the small `t3.micro` size.
- No NAT Gateway was deployed. NAT Gateways incur ongoing hourly and data-processing charges.
- No load balancer, managed database, or other unrelated service was added.
- Public IPv4 addresses and EC2 runtime/storage may still incur charges depending on account eligibility, current AWS pricing, and free-tier terms.

Always check the AWS Pricing Calculator and the account's Billing dashboard for current pricing and actual charges. Free Tier eligibility changes over time and depends on account status and usage.

## Private subnet internet access

The private route table contains only the VPC local route. Private instances can communicate with eligible VPC destinations, but this design does not provide general outbound internet access. If software updates or downloads are required, request approval before adding a NAT Gateway in a public subnet and a corresponding default route in the private route table.

## Cleanup checklist

After the instructor has completed the review/demo, and after confirming that no resources must remain running:

1. Terminate the public and private EC2 test instances.
2. Delete any separately created EBS volumes or snapshots that are no longer needed.
3. Delete the security groups after detaching them from all network interfaces/resources.
4. Disassociate and delete the custom route tables when no longer in use; retain the VPC main route table as AWS requires.
5. Detach and delete the Internet Gateway.
6. Delete the four subnets.
7. Delete the custom VPC.
8. Review Elastic IPs, public IPv4 usage, volumes, snapshots, and the Billing dashboard for remaining chargeable resources.

Delete resources in dependency order. AWS may prevent deletion while dependent resources or network interfaces remain.

## Before cleanup

- Capture the final screenshots and terminal evidence.
- Save the architecture and documentation in GitHub.
- Confirm the instructor does not require a live environment after submission.
- Do not delete resources before the required demonstration if the instructor expects them to remain available.
