# Connecting Your IoT Device to AWS IoT Core: Obtaining a Certificate with Your Private Key's CSR

 &nbsp; This guide will walk you through simulating an IoT device using Node-RED and securely connecting it to AWS IoT Core. We will generate a private key and a Certificate Signing Request (CSR) to obtain an AWS-signed certificate. Leveraging Node-RED, we'll create test flows to mimic IoT device behavior, enabling seamless communication with AWS IoT Core. Get ready to prototype and test your IoT solutions confidently in this hands-on journey! Let's begin!


## OpenSSL Installation

**We start with the installation of OpenSSL first.**

[OpenSSL Download](https://www.openssl.org/source/old/3.1/)

**We go to the website and download the latest version..**


**After downloading OpenSSL, we navigate to the OpenSSL file location, and then enter the "Bin" directory. Using OpenSSL, we create both the private key and the CSR (Certificate Signing Request) in one go with the following code**

```bash
openssl req -newkey rsa:2048 -nodes -keyout domain.key -out domain.csr
```

**If we want it to be generated in an encrypted format, we can use the following code. The difference from the previous one is the absence of the "-nodes" parameter:**


```bash
openssl req -newkey rsa:2048 -keyout domain.key -out domain.csr
```


![](https://i.hizliresim.com/j5kkfz4.jpg)

### We proceed as shown in the figure, then...

![](https://i.hizliresim.com/nx8cw25.jpg)

### We move to the OpenSSL file location, and our Private Key and CSR (Certificate Signing Request) files are generated.













## Node-RED Installation



**We open the Command Prompt.**
```bash
npm install -g --unsafe-perm node-red 
```
**We type the command.**

![](https://i.hizliresim.com/6zfzd68.jpg)

**If you get the output as shown on the screen, it means the installation is successful.**

**Afterward, we open the Command Prompt (cmd).** 
```bash
node-red
```


**We type "node-red" and run Node-RED.**

```bash
http://127.0.0.1:1880/
```

**We can go to "http://localhost:1880" to see that Node-RED is running locally.**

![](https://i.hizliresim.com/npu2rf4.jpg)


### If you encounter a screen like the one below, Node-RED installation is complete:



## Configuring AWS IOT Core


### Creating a Thing
&nbsp; First we need to create a thing to represent our IoT device.
+ &nbsp; Search for the IoT core using the search bar and get into it.
![IoT Core](https://i.hizliresim.com/pjjl0i1.jpg) 
+ &nbsp; To create a thing we need to click on the "things" button under the All Things section from the left navigation bar and then click on the create things button
![2](https://i.hizliresim.com/e2stdum.jpg)
+ &nbsp; Choose to Create single thing and say next
![3](https://i.hizliresim.com/19k4ev2.jpg)
+ &nbsp; Enter the thing name and say next, it is node_thing in our case.
![4](https://i.hizliresim.com/hvc8app.jpg)
+ &nbsp; At Configure device certificate page we have four options. The first option provides us everything we need for connection including private and public keys. We don't want that since we want our private key to be known by only us and not shared with AWS. The next two options require having our own Certificate Authority we also don't interested in that.
&nbsp; Therefore, we choose to skip this step at this time to later get a certificate from the AWS using our certificate signing request file created by our private key. Now our thing is created and ready to use.
![5](https://i.hizliresim.com/pr3m4l4.jpg)

### Creating a Policy
&nbsp; After creating our thing we need to configure a policy for the certificate we will get from AWS.
+ Go to Policies under Security under Manage and then click on create policy.
![6](https://i.hizliresim.com/1i452vq.jpg)
+ Enter the policy name for our case it is node_red_policy
+ Policy effect will be "allow", policy actions will be All AWS IoT actions and lastly policy resource will be All resources by typing "*" for resources.
+ When we say create, our policy will be created and ready to use.
![6](https://i.hizliresim.com/ne94led.jpg)

### Getting a Certificate
&nbsp; Now we will get a certificate from AWS using our CSR file and attach the certificate we got to our thing and policy.
+ Under Manage -> Security -> Certificate section click on Add certificate button and then choose Create certificate.
![7](https://i.hizliresim.com/4izkvpy.jpg)
+ Choose "Create certificate with certificate signing request (CSR)" and upload our CSR file, then choose the status to be active and click on Create button.
![8](https://i.hizliresim.com/tf51n3c.jpg)
+ After creating our certificate AWS will provide us with the certificate and the Certificate Authority file to download. Since we created our private key using rsa we will use the CA 1.
![9](https://i.hizliresim.com/9h5nljp.jpg)
+ After downloading the certificate-related files we will see our created certificate. Now we need to click on our certificate and attach the policy we created and the thing we created to complete our AWS setup. After attaching the two, everything will be ready for the connection. 
![10](https://i.hizliresim.com/lhdp2b8.jpg)
## Configuring Node-Red
&nbsp; We will use node-red to send a string message to the AWS server using MQTT to mimic the behavior of our IoT device.
+ First, we will drag and drop an inject component under the common section and an MQTT out component under the network section from the left side panel.
![11](https://i.hizliresim.com/rqir2zs.jpg)
+ Then we will click on our inject component and change our payload type to string and enter the value we want to send.
+ After that, we will click on the MQTT out component and make our server configuration.
![12](https://i.hizliresim.com/qaje6so.jpg)
+ We can find our server endpoint at AWS under the Settings, Endpoint option. The port number will be default 8883. TLS option will be selected and In the TLS configuration option, the certificate created by AWS, our private key, and, the certificate authority file will be uploaded. After all that, we can choose the topic we want to send the message and our MQTT configuration will be done.
![13](https://i.hizliresim.com/4jlvwme.jpg)
+ Now when we connect the inject component to our MQTT out component and say deploy our node-red setup will be completed. 
![14](https://i.hizliresim.com/25fz7sq.jpg)
+ If everything works fine there should be green box under MQTT that says connected.
![15](https://i.hizliresim.com/3dxy9m8.jpg)

## Listening on AWS and Sending Messages from Node-Red
&nbsp; after completing all the setups that needed to be done we can now listen on aws to the message from our device.
+ Go to the MQTT test client section from the left bar and enter the topic name into the topic filter part. Click on subscribe. In our case, the topic name is kk_test and we are now we are listening to the topic.
+ Go to node-red and click on the button in front of the inject button. 
+ When we go back to the AWS listening screen we should see the message we have sent over node red on the screen.
![16](https://i.hizliresim.com/tepepml.jpg)


### We provide the previously generated certificate, private key, CA files, and AWS endpoint information to the Python client using the following Python code to establish a connection:



```python
from __future__ import print_function
import sys
import ssl
import time
import datetime
import logging, traceback
import paho.mqtt.client as mqtt


endpoint          = "ae32yu86wyxqh-ats.iot.us-east-1.amazonaws.com"
cert              = "b656f742766f382e2ec8f66f7f4fc5749ec9499777c8f0624fabfb6d9892166e-certificate.pem.crt"
key               = "domain.key"
root_ca           = "AmazonRootCA1.pem"
aws_iot_endpoint  = endpoint
url               = "https://{}".format(aws_iot_endpoint)
IoT_protocol_name = "x-amzn-mqtt-ca"


logger      = logging.getLogger()
logger.setLevel(logging.DEBUG)
handler     = logging.StreamHandler(sys.stdout)
log_format  = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
handler.setFormatter(log_format)
logger.addHandler(handler)

def ssl_alpn():
    try:
        #debug print opnessl version
        logger.info("open ssl version:{}".format(ssl.OPENSSL_VERSION))
        ssl_context = ssl.create_default_context()
        ssl_context.set_alpn_protocols([IoT_protocol_name])
        ssl_context.load_verify_locations(cafile=root_ca)
        ssl_context.load_cert_chain(certfile=cert, keyfile=key)

        return  ssl_context
    except Exception as e:
        print("exception ssl_alpn()")
        raise e

if __name__ == '__main__':
    topic = "bang"
    try:
        mqttc = mqtt.Client()
        ssl_context= ssl_alpn()
        mqttc.tls_set_context(context=ssl_context)
        logger.info("start connect")
        mqttc.connect(aws_iot_endpoint, port=443)
        logger.info("connect success")
        mqttc.loop_start()

        while True:
            now = datetime.datetime.now().strftime('%Y-%m-%dT%H:%M:%S')
            logger.info("try to publish:{}".format(now))
            mqttc.publish(topic, now)
            time.sleep(1)

    except Exception as e:
        logger.error("exception main()")
        logger.error("e obj:{}".format(vars(e)))
        logger.error("message:{}".format(e.message))
        traceback.print_exc(file=sys.stdout)
```

### We go to the terminal and.
```bash
python main.py
```
### We run the Python code by typing

![](https://i.hizliresim.com/d2oxm0j.jpg)

### We see that the code is running as shown, and the AWS IoT Client continuously sends messages using a while loop.

![](https://i.hizliresim.com/892kpk3.jpg)

## We see that the messages are coming from Python in this way.

## Conclusion
&nbsp; With the generated private key and CSR, you obtained an AWS-signed certificate for secure communication. Leverage AWS IoT Core's scalability and Node-RED's flexibility to publish and receive messages effortlessly, ensuring effective device communication.

## Contributing

