# Report of the External Services Working Group, part of the Islandora Preservation Interest Group

Ed Fugikawa, Mark Jordan, Danny Lamb, Mark Leggott, Paul Pound, Nick Ruest

December 4, 2014

Currently, Islandora solution packs create derivatives using PHP code (included in Islandora modules) that runs on the Drupal web server, typically on object or datastream ingest or in response to some other event such as a user clicking on a “Regenerate derivatives” link. An alternative approach to creating derivatives is to use external microservices such as those implemented by UPEI. These microservices bypass the derivative creation code included in solution packs by listening to Fedora’s JMS broker for ingest and update events and firing external scripts to create derivatives and ingest them into Fedora.

A subset of the Islandora Preservation Interest Group (the “External Services Interest Group”) proposes a rearchitecting of Islandora so that all derivative creation code is removed from Drupal and reimplemented as external services. These services are controlled by a rules-based workflow engine that responds to events from Drupal that should generate derivatives, such as object and datastream ingestion. The workflow engine then delegates the creation of derivatives to one or more external services, responds to the outcome of the service(s), and acts accordingly by ingesting the output into Fedora as a datastream. Other tasks, such as updating Solr’s index, could also be managed in this way (which would potentially allow us to drop Gsearch from the Islandora stack).

In this architecture, the code within solution packs would need to be repackaged into endpoints within the workflow routing. By default, these endpoints could run on the same server that the Drupal, Fedora, and Solr applications to allow single-server stack installations. However, running the endpoint services on separate servers would allow for the greatest scalability and reusability. The current and proposed architectures are depicted in the following diagram:

![Current and proposed architectures for derivative creation](https://dl.dropboxusercontent.com/u/1015702/linked_to/islandora_derivatives_creation_architectures.png)

A number of workflow engines could satisfy these requirements. UPEI’s microservices currently use the Taverna Workflow management system. An alternative is Apache Camel, which is a rule-based routing and mediation engine that implements standard Enterprise Integration Patterns and that allows for the definition of rules in a variety of domain-specific and general languages. Camel’s advantage is that is offers easy integration with other service-oriented tools such as Apache ServiceMix and Apache ActiveMQ.

Further discussion and research is required to determine the best workflow engine to insert into a rearchitected Islandora stack, or whether it is possible to use more than one workflow engine while allowing standardization on the solution pack and external services ends of the workflow routing. With the advent of Fedora 4 and the certainty that most of the existing Islandora stack needs to be rewritten (at a minimum, the Tuque API and all solution packs), the Islandora community now has an opportunity to review how derivatives are created and how the current binding of derivative-creation code to Drupal modules limits Islandora’s scalability.
