- The ability to handle more requests either by increasing the number of resources (horizontal scaling) or by making one resource bigger or more capable (vertical scaling).

| Horizontal Scaling                                                       | Vertical Scaling            |
| ------------------------------------------------------------------------ | --------------------------- |
| Load balancing required                                                  | NA                          |
| If one machine fails, requests can <br>get routed to another (Resilient) | Single point of failure     |
| Network call (RPC)                                                       | Inter process communication |
| Data inconsistency. Loose transactional<br>guarantee                     | Consistent data             |
| Scales well. Almost linear to throughput.                                | Hardware limit              |
