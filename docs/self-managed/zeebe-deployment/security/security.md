---
id: security
title: "Security"
sidebar_label: "Overview"
description: "This document analyzes Zeebe's security features."
---

Zeebe supports the following security features:

- **[Encrypted client-gateway communication](secure-client-communication.md)** - allows you to secure communication between clients and gateways.
- **[Client authorization](client-authorization.md)** - allows you to supply access credentials to the client so these can be validated by a reverse proxy placed before the gateway.
- **[Encrypted cluster communication](secure-cluster-communication.md)** - allows you to secure communication between all nodes in a cluster