# Network Architecture

KijaniKiosk will use a virtual network divided into public and private
subnets. The public subnet will contain internet-facing resources such
as a load balancer and will have a route to the Internet Gateway.

The private subnet will contain application and data resources that
should not be directly accessible from the internet. It will not have
a direct route to the Internet Gateway. Requests from users will
therefore enter through the public subnet before reaching protected
resources in the private subnet.

This segmentation reduces the attack surface and ensures that sensitive
application and data resources are isolated from direct internet access.

                         INTERNET
                            │
                            │
                    ┌───────▼────────┐
                    │ Internet        │
                    │ Gateway         │
                    └───────┬────────┘
                            │
                  ┌─────────▼─────────┐
                  │   VPC / Network   │
                  │                   │
                  │  ┌─────────────┐  │
                  │  │ Public      │  │
                  │  │ Subnet      │  │
                  │  │             │  │
                  │  │ Load        │  │
                  │  │ Balancer    │  │
                  │  └──────┬──────┘  │
                  │         │          │
                  │         │          │
                  │  ┌──────▼───────┐  │
                  │  │ Private      │  │
                  │  │ Subnet       │  │
                  │  │              │  │
                  │  │ Application  │  │
                  │  │ / Database   │  │
                  │  └──────────────┘  │
                  │                   │
                  └───────────────────┘
