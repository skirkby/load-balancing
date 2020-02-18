# Load Balancing Notes

## What is "load balancing"?

"In computing, load balancing improves the distribution of workloads across multiple computing resources, such as computers, a computer cluster, network links, central processing units, or disk drives."
- https://en.wikipedia.org/wiki/Load_balancing_(computing)

Load balancing is a generic term that essentially means that you take a computing capability and "scale" it across multiple instances of resources. 

Any kind of computing service can benefit from a load balancer. As the Wikipedia definition indicates, that could be computing resources (servers/cpu's, etc.), network resoruces, disk resources, etc. For our purposes, we consider "load balancing" to apply to servers or instances of our service/application.

For example, if you write an application/service in Node.js, you run that application/service on a computer (either physical or virtual). If your app/service is designed properly, you can launch *another* instance of it on *another* server. If your app/service can effectively service 1000 simultaneous users on a single server, then launching a second instance of it on another server brings your supportable audience size up to 2000.

You could give some people the address to the first server, and other people the address to the second server. In that way, you would be trying to balance the potential load across both servers.

But this is not effective. If you give 1000 people the Server1 address, and a different 1000 people the Server2 address, what happens if all 1000 people in group 1 log in?
- you would have 1 server completely maxed out, and another server sitting idle.
- if you have another 500 people who would like to log in, which group do you put them in? You might be tempted to put them in group 2, but what if all 1500 people in group 2 log in the next day (and all 1000 people in group 1 take the day off)?
  - you would have capacity for 2000 logins, and only demand for 1500 logins, but all 1500 logins would try to log into the same server, where you can only handle 1000 logins...
  
How can you efficiently utilize the capacity of all 2000 logins (assuming there will never be a demand for more than 2000 logins)?
And, how can you simplify the advertising of your service with just a single address, instead of trying to keep track of who you gave which address to?

Load balancing is the answer. 

```
     +------+
     | user |
     +--+---+
        |
        v
   +----+-----+
   |   load   |
   | balancer |
   +-----+----+
         |
         v
  +-------------+
  |      |      |
  v      v      v
+-+-+  +-+-+  +-+-+
|svc|  |svc|  |svc|
+---+  +---+  +---+
```

In this way, the only address you need to publish is the load balancer address. 

In this diagram, if each "svc" (service) can handle 1000 logins, then you have a system that can handle 3000 logins simultaneously.

And, the load balancer can ensure that each service is evenly loaded. If 100 people log in, each service will get 33 logins (one will have 34). If all 33 people on server 3 logout, then new logins will be directed by the load balancer to that server until it is balanced.

(This is a simplistic view of how load balancers work, but you get the idea.)

In addition, the load balancer has an ability to check on the "health" of a server. If the server is full, or otherwise "unhealthy" (unable to take new logins), the load balancer can avoid sending new login requests to it. What's more, sophisticated load balancers, working with well-designed services, can take existing logins from an unhealthy server, and move them off onto other servers.

Lastly, modern-day sophisticated load balancers can do more than just direct traffic - they can "horizontally" scale your application by "spinning up" (starting) new instances of your service, and making them available as part of the "service pool".

For example, in the diagram above, you have capacity for 3000 logins. If 2700 people login, and new logins are occurring at a rate of 3 per second, the load balancer can decide that it should start up a 4th server, to handle expected new traffic beyond 3000. You would then automatically be able to handle 4000 logins.

Likewise, if logins slow way down, and logouts begin happening, a sophisticated load balancer can work to "spin down" servers that are not needed. If active logins drop below 2000, and the logout rate is increasing, the load balancer can choose to "spin down" servers 3 and 4.

In today's environment, where servers are often run in the "public cloud" (Amazon Web Services, Google Cloud, Microsoft Azure, and many others), leaving a server running when it's not utilized is just wasted money. So a good load balancer can help with the bottom line...

## What are examples of load balancers?

There are many types of load balancers.

### DNS-based load balancing



### Nginx

One of the most popular is based on the Nginx engine.
- https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/
- https://www.nginx.com/blog/5-performance-tips-for-node-js-applications/

### Homemade load balancer (Node.js and ExpressJS)

I found this article (it's a few years old) about how to build your own load balancer using Node.js and ExpressJS. When you are first learning Node.js and ExpressJS, and this concept of load balancers, this can seem a bit confusing. I'm-a sum up:
- You can use Node.js and ExpressJS to develop a service/application that *needs to be load balanced*.
- You can *also* use Node.js and ExpressJS *to develop a load balancer service* that can be used to load balance other applications.

This article discusses the latter: https://thecodebarbarian.com/building-your-own-load-balancer-with-express-js.html

### AWS / Amazon Elastic Load Balancer (ELB) and Application Load Balancer (ALB) and Network Load balancers (NLB)

Of course, Amazon provides virtual server environments (a service called EC2). If you develop a service that runs on an EC2 instance, the ELB and ALB services can help ensure that incoming requests are balanced across those instances.

Amazon begain with the ELB some time ago (2009), and released the ALB in 2016. This article discusses the difference between them, and provides some guidance on choosing one or the other:
-https://www.sumologic.com/blog/aws-elb-alb/

Neither ELB nor ALB provide "autoscaling" by themselves. They do not add/remove EC2 instances (virtual servers) in response to traffic. Rather, they work together with an EC2 feature called "EC2 Auto Scaling":
- https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html
- https://docs.aws.amazon.com/autoscaling/ec2/userguide/autoscaling-load-balancer.html

EC2 Auto Scaling provides "Auto Scaling Groups", to/from which new EC2 instances with your service/app can be automatically added/removed. ELB/ALB/NLB can be configured to work with Auto Scaling (see the documentation).

You can find documentation on AWS "Elastic Load Balancing" (which includes documentation for Application and Network Load Balancing as well):
- https://docs.aws.amazon.com/elasticloadbalancing/index.html

Other resources:
- https://cloudacademy.com/blog/application-load-balancer-vs-classic-load-balancer/
  - https://youtu.be/uTrpk-atNFc
- https://medium.com/containers-on-aws/using-aws-application-load-balancer-and-network-load-balancer-with-ec2-container-service-d0cb0b1d5ae5

### Microsoft Azure Load Balancing

I am not at all familiar with this service, but it is conceptually the same as the AWS load balancer, and other load balancers. I include it here to demonstrate the ubuquity of these kinds of services.

- https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-overview
- https://azure.microsoft.com/en-us/services/load-balancer/
- https://www.youtube.com/results?search_query=azure+load+balancer

### Google Cloud Load balancing

Similarly, I am not super-familiar with Google's load balancing service, but I understand that it *does* include autoscaling (autoscaling isn't a separate product):

- https://cloud.google.com/load-balancing
- https://cloud.google.com/load-balancing/docs
- https://cloud.google.com/load-balancing/docs/concepts
- https://cloud.google.com/load-balancing/docs/load-balancing-overview
- https://www.youtube.com/watch?v=HUHBq_VGgFg


### Hardware load balancers (i.e. F5)

In addition to cloud-based services like AWS, Azure, and Google, as well as software-based load balancers like Nginx, you will find purpose-built load balancers in the marketplace.

One popular load balancer is a product from F5 Networks:
- https://www.f5.com/services/resources/glossary/load-balancer
- https://www.f5.com/services/resources/white-papers/load-balancing-101-nuts-and-bolts

You can learn about "layer 7" and "layer 4" load balancing, both of which are related to "the OSI model":
- https://en.wikipedia.org/wiki/OSI_model

The OSI model divides application and service design and communication into "layers":

- Layer 1 - Physical : in a wired Ethernet network, this would be the copper. In a wireless network, this would be the wireless signal.
- Layer 2 - Data Link Layer : the parts of the network that connect and control communication between two devices. This would include network cards in the devices, and routers or hubs in between them.
- Layer 3 - Network Layer : in a TCP/IP network, this almost always refers to the IP protocol (Internet Protocol), which is the lowest level protocol in the TCP/IP stack. (There are other Layer 3 protocols in the TCP/IP stack, but they are not as well-known.)
- Layer 4 - Transport Layer : In a TCP/IP network, this almost always refers to the TCP or UDP protocols (Transmission Control Protocol or User Datagram Protocol). Most of the higher-level protocols that you are aware of are transmitted inside a TCP or UDP packet (which, in turn, is transmitted inside an IP packet). For example, HTTP, SMTP, and FTP are all higher-level protocols that use TCP and/or UDP.
  - note that the TCP/IP protocol stack is not a strict OSI-modeled stack, so parts of the TCP/IP protocol suite don't fit neatly within the OSI model. TCP is one such protocol. Parts of TCP fit in Layer 4, and parts fit in Layer 5. So I typically think of TCP as an OSI Layer 4/5 element. UDP, however, is squarely in Layer 4, because it is "connectionless" as opposed to "connection-oriented". (Connectionless and connection-oriented are kind of like "stateful" and "stateless".)
- Layer 5 - Session Layer : Controls the dialoge between computers (which are session-based). TCP fits in this layer (though parts of it fit in Layer 4).
- Layer 6 - Presentation Layer : This is an esoteric layer that has more to do with data encoding and the transmission of data in an agnostic format.
- Layer 7 - Application Layer : This represents the part of the system where logic resides, independent of communication protocols or data formats. This is the service or app. Protocols used to communicate directly with the application's logic (different from protocols meant to transmit requests/responses to the application's location) are referred to as "application protocols", and fall in layer 7.
  - HTTP, SMTP, FTP, and other Internet application protocols fall into this layer.

Layer 7 load balancers sit in between two systems using an application protocol and act as a proxy to each side of the "conversation". In this way, Layer 7 load balancers can route application protocols (layer 7 protocols like HTTP) to various end points in order to achieve load balancing.

Layer 4 load balancers don't look at application protocols like HTTP, but rather look at transmission protocols like TCP to determine routing decisions based on load.

Both software and hardware based load balancers, as well as commercial and open source load balancers, often support either Layer 4 or Layer 7 load balancing.

You will also hear "layer 4" and "layer 7" applied to firewalls and other proxies (sometimes called "application firewalls", as opposed to "network firewalls").

Other resources:
- https://www.haproxy.com/blog/loadbalancing-faq/
- https://freeloadbalancer.com/load-balancing-layer-4-and-layer-7/


-end- (for now)



