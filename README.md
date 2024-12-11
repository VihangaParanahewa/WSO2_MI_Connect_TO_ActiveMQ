# WSO2_MI_Connect_TO_ActiveMQ

This for the necessary configurations for connect WSO2MI with the ActiveMQ.

## Connecting to Active MQ
 - Copy the following client libraries from the `ACTIVEMQ_HOME/lib` directory to the `MI_HOME/lib` directory
   https://mi.docs.wso2.com/en/latest/install-and-setup/setup/brokers/configure-with-activemq/
 - If you want the Micro Integrator to receive messages from an ActiveMQ instance, you need to update the deployment.toml file with the relevant connection parameters.
```bash
[[transport.jms.listener]]
name = "myTopicListener"
parameter.initial_naming_factory = "org.apache.activemq.artemis.jndi.ActiveMQInitialContextFactory"
parameter.provider_url = "tcp://localhost:61616"
parameter.connection_factory_name = "TopicConnectionFactory"
parameter.connection_factory_type = "topic"
parameter.cache_level = "consumer"
parameter.username = "system"
parameter.password = "manager"
```
- When configuring the JMS transport with ActiveMQ, you can append ActiveMQ-specific properties to the value of the parameter.provider_url property. For example, you can set the redeliveryDelay and initialRedeliveryDelay properties when configuring a JMS inbound endpoint as follows:
```bash
parameter.provider_url = "tcp://localhost:61616?jms.redeliveryPolicy.redeliveryDelay=10000&amp;jms.redeliveryPolicy.initialRedeliveryDelay=10000"
```
- The above configurations do not address the problem of transient failures of the ActiveMQ message broker. For example, if the ActiveMQ goes down and becomes active again after a while, the Micro Integrator will not reconnect to ActiveMQ. Instead, an error will be thrown until the Micro Integrator is restarted.
  To avoid this problem, you need to add the following value. his simply makes sure that reconnection takes place. The failover prefix is associated with the Failover transport of ActiveMQ.
```bash
parameter.provider_url = "failover:tcp://localhost:61616"
```

- Remember to add following JMS parameters to the proxy service configuration
- When configuring the jms listener, be sure to add the connection factory service-level jms parameter to the synapse configuration with the name of the already defined connection factory.
```bash
<parameter name="transport.jms.ConnectionFactory">myTopicListener</parameter>
<parameter name="transport.jms.DestinationType">topic</parameter>
<parameter name="transport.jms.Destination">TestTopic</parameter>
```

- Optional configurations for proxy service configurations
```bash
<parameter name="redeliveryPolicy.maximumRedeliveries">3</parameter>
<parameter name="redeliveryPolicy.redeliveryDelay">2000</parameter>
<parameter name="transport.jms.SessionTransacted">true</parameter>
<parameter name="transport.jms.CacheLevel">consumer</parameter>
```