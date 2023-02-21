I think we should separate two distinct use cases:

1. Contract Agreement non-repudiation - proof that an IDS Dataspace Protocol Contract Agreement was signed by a counter-party.
2. Data transfer non-repudiation of data transfer - proof that specific data was received by a counter-party.

I have created a design for IDS-based Contract Agreement non-repudiation of here:

https://github.com/Metaform/ds-contract

## Data Transfer Non-Repudiation

Data transfer non-repudiation has much stricter quality of service requirements than certified message delivery since the latter only requires proof that a message was received and
not proof that particular message **content** was received.

Consider two scenarios.

### Scenario I: Third-Party Escrow Data Transfer Non-Repudiation

A third party escrow system could be used. In this scenario, the provider registers a signed and encrypted message with the escrow system. Upon registration, the escrow system
notifies the counter-party the message is available and records access. This approach has numerous downsides:

1. The escrow system is a single point of failure
2. Ensuring qualities of service such as reliable delivery and security would be extremely challenging at the scale required by Catena-X. At a minimum, geographic fail-over of all
   message stores would be required.
3. Numerous business questions would arise. For example, which entity is responsible if the escrow service fails to notify a counter-party due to a technical error?
4. Contract confidentiality can potentially be compromised because it would be possible to trace transactions between parties.

### Scenario II: Two-Party Data Transfer Non-Repudiation

This scenario is not technically feasible, but as a theoretical exercise, consider the following implementation where data is transferred synchronously using a JSON-LD message:

1. The provider creates a JSON-LD message containing a JWS of the message content hash
   using [RDF Dataset Canonicalization](https://www.w3.org/community/reports/credentials/CG-FINAL-rdf-dataset-canonicalization-20221009/).
2. The provider opens a transaction, TX-P1, against its datastore.
3. The provider sends the message contents and the JWS to the consumer.
4. The consumer opens a transaction, TX-C1, as it receives the message,
5. The consumer verifies the JWS and hash
6. The consumer creates a hash of the provider JWS, sends it to a third-party registry service as described in [SAMO](https://github.com/Metaform/ds-contract) and stores a receipt
   and the message to its datastore.
7. The consumer creates a JWS from the hash of the received message and returns it to the provider.
8. The consumer commits its transaction, TX-C1.
9. The provider verifies the consumer JWS.
10. The provider creates a hash of the conumer JWS, sends it to a third-party registry service as described in [SAMO](https://github.com/Metaform/ds-contract) and stores a receipt
    and the message to its datastore.
11. The provider commits its transaction, TX-P1.

## A Simpler Approach

If the required qualities of service can be relaxed, a much simpler approach to data transfer verification can be implemented. Consider the following, which relies on audit logs:

1. The provider creates a message.
2. The provider opens a transaction, TX-P1.
3. The provider sends the message to the consumer.
4. The consumer opens a transaction, TX-C1.
5. The consumer creates an audit entry in its transactional store containing the contents of the message.
6. The consumer commits the TX-C1 transaction and sends an ACK to the provider.
7. The provider receives the ACK and creates an audit entry in its transactional store containing the contents of the message.
8. The provider commits its TX-P1 transaction.

In this scenario, the audit entries could be used as evidence in disputes. If required, the integrity of the audit stores could be monitored by an external party. In addition,
message signing could be included in this approach. 

