![green grass](fresh-small.jpg)

# A fresh approach to fighting bots

The biggest challenge faced by outdated bot prevention methods, which tend to focus on "server interactions" – 
the requests that bots make to your server – is the dearth of information that can effectively identify and 
differentiate between humans and bots. Fraudsters are well aware of this and have learnt how to spoof various 
Application Programming Interface (API) calls to the servers, circumventing many forms of bot detection and 
prevention.  

This is why a more sophisticated approach has been devised that does not necessitate tight integration with 
particular game logic and makes use of sensor data – information that is difficult to fake. 

A baseline of authentic user behavior on specific flows within an app is built by leveraging sensor data that 
uses information on touch pressure and position, accelerometer, gyroscope, light sensors and so on to track 
everything that happens to a device or its peripherals. The technology then monitors all the interactions with 
the mobile game through several elements, such as how fast the clicks happen and where they are taking place 
on the screen, the angle of the touch, device’s position and movement.

By comparing those interactions with what is considered normal for that specific game, players that are 
operating outside the parameters through anomaly detection will be highlighted.

## The importance of data processing efficiency in detecting bots

To make the bot detection technology work effectively, the behavioral data is collected on the client side and 
transmitted to our servers for further analysis.

It is where our Unbotify solution analyzes fine-grained data points (touch and sensor interactions). The 
amount of data is highly game-dependent. For instance, this kind of data is extremely rich on mobile devices 
because there are more sensors available to gather data from. It can amass a constant stream of one data 
packet per second per player, capturing the interaction details at millisecond precision. This can add up to 
terabytes per day even for a game with a modest 50.000 concurrent players.

It therefore requires the ingestion and management of vast amounts of data to ensure high accuracy and swift 
response in detecting botting activity and cheating, thus maintaining player community integrity. Efficiency 
in data processing is crucial for us to keep an eye on more players more frequently. Otherwise, only a portion 
of players can be focused on and important interactions could be overlooked.

## Data processing efficiency is one of the keys to fight bots

Unbotify filters through the large amounts of player data with the help of cloud-based resources. This also 
allows us to scale according to the developer’s demand.

Unbotify uses Databricks to orchestrate the data processing. We’d like to share some insights into how this is 
done effectively and efficiently. A series of blog post installments discusses how to address data processing 
bottlenecks. The findings demonstrate that optimized data partitioning, shuffling, and serialization can 
significantly enhance performance and cost efficiency, ultimately enabling scalable, efficient data processing 
pipelines.

# Outline

Here’s an outline of the series: 

1. [A fresh approach to fighting bots](fresh.md) (this article)
2. [Background: Databricks and Apache Spark](background.md)
3. [Key Performance Features in Spark](perf.md)
4. [Case study 1: Bottlenecks in Traditional Data Partition Management](case1.md)
5. [Case Study 2: Optimizing Data Loading Workflows](case2.md)
6. [Case Study 3: Window Functions as a Bottleneck in Data Aggregation](case3.md)
7. [Case Study 4: Transitioning from User Defined Functions to Scalable Spark-Based Solutions](case4.md)
8. [Case Study 5: Reduction in DataFrame Operations](case5.md)
9. [Conclusion: A fair gaming environment needs a cutting-edge bot protection](conclusion.md)
