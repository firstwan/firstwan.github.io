---
title: Event sourcing with Apache Kafka
author: first_wan
categories: [Programing]
tags: [DevOps, Kafka]
pin: false
---

## Introduction
Apache Kafka is a distributed streaming system primary used to build real-time streaming data pipelines and applications that adapt to the data streams. It combines messaging, storage, and stream processing to allow storage and analysis of both historical and real-time data. In this article, we going to build a simple application that produce and consumer the data from Kafka

![kafka_practice.png](/blogs/kafka_practice/overview_1.png)

We will use Confluent to configure our Kafka and create web application to act as Producer to produce event message and a console app to retrieve the event message by subscribe to Kafka.

## Walkthrough

### Create a Confluent account

You need to create Confluent account for this exercise. If you already have it, you can skip to next step.

Create an account on [Confluent](https://confluent.cloud/signup)

![kafka_practice.png](/blogs/kafka_practice/account_1.png)

You will be request to key in payment information, but don’t worry it won’t charge you a cent as long as you clean up all the resources after you finish this exercise

#### Apply Promo code

There are few promo code you can apply to get free usage.

- Navigate to “Billing & payment”
    
    ![kafka_practice.png](/blogs/kafka_practice/account_2.png)
    
- Navigate to “Payment details & contacts”
    
    ![kafka_practice.png](/blogs/kafka_practice/account_3.png)
    
- Add new promo code at the bottom of the page

    ![kafka_practice.png](/blogs/kafka_practice/account_4.png)

### Create Cluster

Before we can produce or consume event from Kafka, we need to create a cluster. Clusters are a group of interconnected Kafka brokers that work together to manage and distribute the data streams. 

1. Click “Add Cluster”
    
    ![kafka_practice.png](/blogs/kafka_practice/cluster_1.png)
    
2. Select the environment you want to use, we will have a **“default”** environment when you create your Confluent account.
    
    ![kafka_practice.png](/blogs/kafka_practice/cluster_2.png)
    
3. Enter a name for cluster, and choose “Basic” type.
    
    ![kafka_practice.png](/blogs/kafka_practice/cluster_3.png)
    
4. You can select any region you want.

    ![kafka_practice.png](/blogs/kafka_practice/cluster_4.png)

### Create API key

We need an API key to connect our application and Confluent Kafka.

1. Navigate to **“API Keys”** and create a key.
    
    ![kafka_practice.png](/blogs/kafka_practice/apikey_1.png)
    
2. We will select create an account API key for this walkthrough.
    
    ![kafka_practice.png](/blogs/kafka_practice/apikey_2.png)
    
3. Confluent will generate the API key and secret, click **“Download and continue”** will download the API key as text file for later use.

    ![kafka_practice.png](/blogs/kafka_practice/apikey_3.png)

### Create Topic

Topics are categories used to organize messages, producer will publish their messages to topics and consumer subscribe topics to get the messages data. 

1. Navigate to **“Topics”** and create a topic
    
    ![kafka_practice.png](/blogs/kafka_practice/topic_1.png)
    
2. Give the topic a name.
    
    > Note that the topic name is used on producer and consumer, if you choose a name different with mine, you need to put your on producer and consumer later.
    {: .prompt-tip }

    
    ![kafka_practice.png](/blogs/kafka_practice/topic_2.png)
    
3. Data contract is a service that let you define the data structure of your topic. You can learn more from [here](https://docs.confluent.io/cloud/current/sr/schemas-manage.html?ajs_aid=86d13ce3-deeb-48c7-80da-b15f8b6df36b&ajs_uid=5684924#create-a-topic-schema). We can click **“Skip”** for this walkthrough.

    ![kafka_practice.png](/blogs/kafka_practice/topic_3.png)

### Create Producer

A producer will produce the message, which actually just send a message to Kafka. I will create a simple Blazor web app for this walkthrough, but an simple API that contain an endpoint to send message to Kafka also will achieve the same result.

1. Create Blazor web app

    ![kafka_practice.png](/blogs/kafka_practice/producer_1.png)

2. Name the project as **“KafkaProducer”**.
    
    > I didn’t place the solution and project in same directory because I will create a new project for consumer in same solution later.
    {: .prompt-tip }

    ![kafka_practice.png](/blogs/kafka_practice/producer_2.png)

3. To keep it simple, Select **“Server”** render mode.

    ![kafka_practice.png](/blogs/kafka_practice/producer_3.png)

4. Add the Kafka config into `appsetting.json`{: .file-path}
    
    ![kafka_practice.png](/blogs/kafka_practice/producer_4.png)
    
    - **API key & secret:** The API key from previous step.
    - **BootstrapServers:** You can get the bootstrap server from Confluent > Cluster > Cluster Settings
        
        ![kafka_practice.png](/blogs/kafka_practice/producer_5.png)
        
5. Right click the “Dependencies” on the right panel and click “Manage NuGet Packages”

    ![kafka_practice.png](/blogs/kafka_practice/producer_6.png)

6. Click “Browse”, enter “kafka” in the text box, install the Confluent.Kafka

    ![kafka_practice.png](/blogs/kafka_practice/producer_7.png)

7. Click “Apply”
    
    ![kafka_practice.png](/blogs/kafka_practice/producer_8.png)
    
8. We will use the `Home.razor`{: .file-path} be our producer page.

    ![kafka_practice.png](/blogs/kafka_practice/producer_9.png)

9. Remove the content inside `Home.razor`{: .file-path} and replace it with below
    
    ```csharp
    @page "/"
    @rendermode InteractiveServer
    @using Confluent.Kafka
    @inject IConfiguration Configuration
    
    <PageTitle>Producer</PageTitle>
    
    <h3>Producer</h3>
    
    <div class="text-center flex-fill d-flex justify-content-center align-items-center">
        <div>
            <label for="message">Enter Text:</label>
            <input type="text" id="message" name="message" @bind="newMessage"/>
        </div>
        <div>
            <button @onclick="ProduceMessage">Submit</button>
        </div>
    </div>
    ```
    
    This is just a simple HTML with one text box for user input any message and a button to send the message to Kafka. Next we will be adding the actual login that produce the message.
    
10. First we need to read the Kafka config from `appsetting.json`{: .file-path}
    
    ```csharp
    private IConfiguration configuration;
    
    protected override void OnInitialized()
    {
        configuration = Configuration.GetSection("Kafka");
    }
    ```
    
11. Next, we need to have a function to produce the message
    
    ```csharp
    private string? newMessage;
    
    private void ProduceMessage()
    {
        string topic = configuration["Topic"]!;
    
        ProducerConfig config = new()
            {
                BootstrapServers = configuration["BootstrapServers"],
                SecurityProtocol = SecurityProtocol.SaslSsl,
                SaslMechanism = SaslMechanism.Plain,
                SaslUsername = configuration["ApiKey"],
                SaslPassword = configuration["ApiSecret"],
            };
    
        // creates a new Kafka producer instance
        using (var producer = new ProducerBuilder<Null, string>(config).Build())
        {
            // produces the message to kafka
            producer.Produce(topic, new Message<Null, string> { Value = newMessage! },
              (deliveryReport) =>
              {
                  if (deliveryReport.Error.Code != ErrorCode.NoError)
                  {
                      Console.WriteLine($"Failed to deliver message: {deliveryReport.Error.Reason}");
                  }
                  else
                  {
                      Console.WriteLine($"Produced event to topic {topic}: key = {deliveryReport.Message.Key,-10} value = {deliveryReport.Message.Value}");
                  }
              });
            producer.Flush(TimeSpan.FromSeconds(10));
        }
    
        newMessage = null;
    }
    ```
    
    We feed the config from `appsetting.json`{: .file-path} into `ProducerConfig` and build a Kafka producer instance to produce the message. `Flush()` method will send all the message in the internal queue to Kafka.
    
    > Note: This might not the actual Kafka producer setup, the producer will only send out message if the message in the internal queue ≥ `queue.buffering.max.ms` . Read more from [here](https://github.com/confluentinc/confluent-kafka-python/issues/137).
    {: .prompt-tip }
    
12. Below is the combined HTML code and logic
    
    ```csharp
    @page "/"
    @rendermode InteractiveServer
    @using Confluent.Kafka
    @inject IConfiguration Configuration
    
    <PageTitle>Producer</PageTitle>
    
    <h3>Producer</h3>
    
    <div class="text-center flex-fill d-flex justify-content-center align-items-center">
        <div>
            <label for="message">Enter Text:</label>
            <input type="text" id="message" name="message" @bind="newMessage" />
        </div>
        <div>
            <button @onclick="ProduceMessage">Submit</button>
        </div>
    </div>
    
    @code {
        private string? newMessage;
        private IConfiguration configuration;
    
        protected override void OnInitialized()
        {
            configuration = Configuration.GetSection("Kafka");
        }
    
        private void ProduceMessage()
        {
            string topic = configuration["Topic"]!;
    
            ProducerConfig config = new()
                {
                    BootstrapServers = configuration["BootstrapServers"],
                    SecurityProtocol = SecurityProtocol.SaslSsl,
                    SaslMechanism = SaslMechanism.Plain,
                    SaslUsername = configuration["ApiKey"],
                    SaslPassword = configuration["ApiSecret"],
                };
    
            // creates a new Kafka producer instance
            using (var producer = new ProducerBuilder<Null, string>(config).Build())
            {
                // produces the message to kafka
                producer.Produce(topic, new Message<Null, string> { Value = newMessage! },
                  (deliveryReport) =>
                  {
                      if (deliveryReport.Error.Code != ErrorCode.NoError)
                      {
                          Console.WriteLine($"Failed to deliver message: {deliveryReport.Error.Reason}");
                      }
                      else
                      {
                          Console.WriteLine($"Produced event to topic {topic}: key = {deliveryReport.Message.Key,-10} value = {deliveryReport.Message.Value}");
                      }
                  });
                producer.Flush(TimeSpan.FromSeconds(10));
            }
    
            newMessage = null;
        }
    }
    ```
    

#### Example Output

![kafka_practice.png](/blogs/kafka_practice/producer_10.png)

### Create Consumer

Consumer will subscribe a topic and keep pooling the message data from topic. We will create a simple console application as Kafka Consumer.

1. Right click the solution and create new project
    
    ![kafka_practice.png](/blogs/kafka_practice/consumer_1.png)
    
    ![kafka_practice.png](/blogs/kafka_practice/consumer_2.png)
    
2. Select **“Console App”**
    
    ![kafka_practice.png](/blogs/kafka_practice/consumer_3.png)
    
3. Name it as **“KafkaConsumer”**
    
    ![kafka_practice.png](/blogs/kafka_practice/consumer_4.png)
    
4. Leave other as default
    
    ![kafka_practice.png](/blogs/kafka_practice/consumer_5.png)
    
5. Same as KafkaProducer, install Confluent.Kafka dependency
    
    ![kafka_practice.png](/blogs/kafka_practice/consumer_6.png)
    
6. Now we can implement the consumer logic. First we need to prepare the config value of the consumer.
    
    ![kafka_practice.png](/blogs/kafka_practice/consumer_7.png)
    
    - **BoostrapServer**, **ApiKey**, and **ApiSecret** is same as Producer
    - **groupId:** Is string that represent a consumer group, so that a message won’t process by different consumer node in the same group
7. Prepare the ConsumerConfig
    
    ```csharp
    ConsumerConfig config = new()
    {
        GroupId = groupId,
        AutoOffsetReset = AutoOffsetReset.Earliest,
        BootstrapServers = bootstrapServer,
        SecurityProtocol = SecurityProtocol.SaslSsl,
        SaslMechanism = SaslMechanism.Plain,
        SaslUsername = apiKey,
        SaslPassword = apiSecret,
    };
    ```
    
8. Create consumer instance to read message from topic
    
    ```csharp
    using (var consumer = new ConsumerBuilder<string, string>(config).Build())
    {
        consumer.Subscribe(topic);
        while (true)
        {
            // consumes messages from the subscribed topic and prints them to the console
            var cr = consumer.Consume();
    
            try
            {
                Console.WriteLine($"Consumed event from topic \"{topic}\": value = {cr.Message.Value}");
            }
            catch (Exception ex)
            {
                Console.WriteLine(ex.Message);
            }
    
        }
    }
    ```
    
9. Below is the combined consumer logic
    
    ```csharp
    using Confluent.Kafka;
    
    Console.WriteLine("Consumer Starting");
    
    string topic = "order";
    string groupId = "kitchen-group-1";
    string bootstrapServer = "pkc-921jm.us-east-2.aws.confluent.cloud:9092";
    string apiKey = "U2CFPHN5KWCOONF3";
    string apiSecret = "9SivBcQfQRHt1+XQL9dXH+0gWVOGL5CyNNLSYrQWkoX8T2rF128EsXQZRFHgsKqY";
    
    ConsumerConfig config = new()
    {
        GroupId = groupId,
        AutoOffsetReset = AutoOffsetReset.Earliest,
        BootstrapServers = bootstrapServer,
        SecurityProtocol = SecurityProtocol.SaslSsl,
        SaslMechanism = SaslMechanism.Plain,
        SaslUsername = apiKey,
        SaslPassword = apiSecret,
    };
    
    // creates a new consumer instance
    using (var consumer = new ConsumerBuilder<string, string>(config).Build())
    {
        consumer.Subscribe(topic);
        while (true)
        {
            // consumes messages from the subscribed topic and prints them to the console
            var cr = consumer.Consume();
    
            try
            {
                Console.WriteLine($"Consumed event from topic \"{topic}\": value = {cr.Message.Value}");
            }
            catch (Exception ex)
            {
                Console.WriteLine(ex.Message);
            }
    
        }
    }
    
    ```
    

#### Example output
![kafka_practice.png](/blogs/kafka_practice/consumer_8.png)

### Extra: Start Producer and Consumer at the same time

Since the Producer and Consumer is in different project, you can only execute one project by default. But Visual Studio allow to configure execute both project in the same time.

1. Right click the solution and select **“Properties”**
    
    ![kafka_practice.png](/blogs/kafka_practice/extra_1.png)
    
    ![kafka_practice.png](/blogs/kafka_practice/extra_2.png)
    
2. Select **“Multiple startup project”** and change action to **“Start”** for both project. Then click **“Apply”** and **“Ok”**
    
    ![kafka_practice.png](/blogs/kafka_practice/extra_3.png)
    
3. Select **“Yes”** in popup window
    
    ![kafka_practice.png](/blogs/kafka_practice/extra_4.png)
    
4. Now you can start boh projects with **“Start”**

    ![kafka_practice.png](/blogs/kafka_practice/extra_5.png)

    ![kafka_practice.png](/blogs/kafka_practice/extra_6.png)

## Clean up

To unexpected charges from Confluent, we need to delete the resources we created in confluent.

1. Navigate to Cluster > Cluster settings
    
    ![kafka_practice.png](/blogs/kafka_practice/clean_1.png)
    
2. At the bottom of cluster settings click the **“Delete cluster”** button
    
    ![kafka_practice.png](/blogs/kafka_practice/clean_2.png)
    
3. Enter the cluster name
    
    ![kafka_practice.png](/blogs/kafka_practice/clean_3.png)
    
4. Done

    ![kafka_practice.png](/blogs/kafka_practice/clean_4.png)

## Conclusion

You had successfully created a Kafka Producer that produce event messages into Apache Kafka and a Kafka Consumer that subscribe the Apache Kafka to consume event messages. Understand the relationship between the components of the Apache Kafka like cluster, topics, producer, and consumer allow you to build a real-time event streaming application. Take note that this is just a simple Apache Kafka application, there are still a lot of component not cover like schema registry, data connector, and etc.

## References
- [Learn Apache Kafka® & Flink®](https://developer.confluent.io/courses/#fundamentals)
- [Kafka InvalidSessionTimeoutError](https://github.com/dpkp/kafka-python/issues/837)
- [WebAssembly Journey, [HANDS ON] real-time web application with Blazor + Kafka + SignalR](https://sercandumansiz.medium.com/webassembly-journey-blazor-kafka-signalr-f76bce892723)
- [Kafka is Flush a must ?](https://github.com/confluentinc/confluent-kafka-python/issues/137)
- [Kafka producer difference between flush and poll](https://stackoverflow.com/questions/52589772/kafka-producer-difference-between-flush-and-poll)
- [understanding consumer group id](https://stackoverflow.com/questions/41376647/understanding-consumer-group-id)
- [What is Kafka? - Apache Kafka Explained - AWS](https://aws.amazon.com/what-is/apache-kafka/)
