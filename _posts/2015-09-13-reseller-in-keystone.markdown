---
layout: post
title: Reseller use case in Keystone
date: '2015-09-13 17:48:11'
---

The OpenStack Identity Service (Keystone) has been on a journey to evolve its capability to support increasingly advanced cloud management models. From “one customer, one project” (Folsom and earlier), to “Customers with many projects” (Grizzly) to “Cloud, Domain & Project Admins” (Icehouse, Juno), to “Hierarchical Projects” in Kilo. 
Hierarchical Projects, removed the flat project structure inside of a domain and enabled users to create sub-projects that might better reflect its organizational structure. This concept is detailed by Raildo Mascena at: http://raildo.me/hierarchical-multitenancy-in-openstack/

We are now taking it further. We are going to support a new model where Cloud Providers can allow resellers to manage their own part of a cloud, enabling them to manage their customers who have their own users and groups without having the Cloud Provider in the loop.

## A brand new kind of project

Until Liberty, Keystone held two distinguished concepts for resource management. The **domains**, that are containers of projects, users and groups, and the **projects**, that are containers of OpenStack resources, such as virtual machine instances, images and usage quotas.

With the concept of Reseller, the domain entity will be redefined. Domains will become a special type of project. We will have, the regular projects, as in Kilo, and the **projects that act as domains**, a.k.a. the domain-powered projects. These new projects that act as domains are capable of holding regular projects resources as well as the former domain resources, and, unlike former domains, can be created at any point of the hierarchy, instead of only at the hierarchy top level.

Hierarchical Multitenancy allowed us to have the following structure:
![Old structure](/content/images/2015/09/old.png)
In this scenario, Marketing, a project in domain Coca-cola shares the same users of Coca-cola. In a big department, it is desirable to have the possibility of managing its own set of users, instead of having them attached to its domain. With the addition of the Reseller concept, this case will be possible, as follows: 
![New structure](/content/images/2015/09/new.png)

Marketing is now a project that acts as a domain, child of Coca-cola, that is also a super-powerful project. Both of them are able to create users isolated from each other. In the other hand, Sales project keeps the same behaviour, with its users attached with Coca-cola.

This resource isolation can go deeper, if Marketing intends to create a sub-project that acts as a domain, and then give this piece of its resources to, for example, a third party department that is going to be responsible for a project and has a different set of users.

Another improvement brought by reseller is the possibility of implicitly setting quota for a domain. This is a benefit of projects acting as domains approach, that, together with Nested Quota implementation (available in Liberty in Cinder, and in Mitaka in Nova) will allow a better hierarchy control from parents.

Thank you all on Distributed Systems Lab and the keystone core team for the support during this cycle.

See you in Tokyo for Mitaka :)
さようなら