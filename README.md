# Getting Started with AWS IoT on Intel Edison

![Intel Edison + AWS IoT](https://cloud.githubusercontent.com/assets/2881361/10375573/439555c4-6dc7-11e5-8eb0-75b9f1506f30.png)

## Prepare Your Intel Edison

Get started with your Intel Edison by updating to the latest firmware
and setting up a serial terminal. You can find the instructions here:

[Get started with Intel Edison technology](https://software.intel.com/en-us/iot/library/edison-getting-started)

## Prepare Your Intel Edison for AWS IoT

Before you continue, we recommend that you read the AWS IoT
[Quickstart](http://docs.aws.amazon.com/iot/latest/developerguide/iot-quickstart.html).
If you are already familiar with AWS IoT, continue to the next step.

### Install the AWS CLI

The AWS CLI is used to interoperate with Amazon Web Services.  To view
the help documentation, you must install Groff and a different version
of less.

We assume you have internet access from the device to download and
install these packages.

### Install the Python Package Manager (pip)

Setup the Edison repo by replacing anything you have in the /etc/opkg/base-feeds.conf file with the following
   ``` bash
   src/gz all http://repo.opkg.net/edison/repo/all
   src/gz edison http://repo.opkg.net/edison/repo/edison
   src/gz core2-32 http://repo.opkg.net/edison/repo/core2-32
   ```

Setup the IoT repo and apply the new configurations.
   ``` bash
   echo "src intel-iotdk http://iotdk.intel.com/repos/3.0/intelgalactic/opkg/i586" > /etc/opkg/intel-iotdk.conf
   opkg update
   ```

Install pip with setuptools.
   ``` bash
   wget https://bootstrap.pypa.io/get-pip.py --no-check-certificate 
   python get-pip.py
   wget --no-check-certificate https://bootstrap.pypa.io/ez_setup.py
   python ez_setup.py --insecure
   ```

### Install the AWS CLI

   Install `man` to also install `groff` and `less`, which is required
   by `aws help`.
   
   ``` bash
   opkg install man
   ```

   Install the AWS CLI by issuing the following command:

   ``` bash
   $ pip install awscli
   ``` 

## Setting Up Your Edison as an AWS IoT Thing

### Get AWS Credentials

The AWS CLI is now installed. Create a new IAM user and get API
credentials from the AWS Management Console by following the steps in
[Getting Set Up with the AWS Command Line Interface](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-set-up.html#cli-signup). After
you have an access ID and key, configure the AWS CLI credentials by
issuing the following command:

   ``` bash 
   $ aws configure 
   ```

**NOTE:** To configure AWS IoT, ensure that your default region is set
  to a region where AWS IoT is available such as `us-east-1`. The
  default output format can be left as JSON.

In order to get permission to download the AWS IoT tools, attach the
administrator account policy to the user. In the IAM console, in the
Users panel, select the user you created, attach the policy, and then
select the administrator account.

### Generate Certificates

1. Create a folder where your certificates will be stored:

   ``` bash
   $ cd ~
   $ mkdir aws_certs
   $ chmod 700 aws_certs
   $ cd aws_certs
   ```

2. A client certificate must be generated to authenticate to the AWS IoT
  topic with MQTT. Run the following to create the certificate:

   ``` bash
   $ aws iot create-keys-and-certificate               \
        --set-as-active                                 \
        --certificate-pem-outfile my_certificate.pem    \
        --public-key-outfile my_public_key.pem          \
        --private-key-outfile my_private_key.pem
   ``` 

3. At the top of the output, locate the `certificateArn` property.
Copy the value, which has a pattern of
`arn:aws:iot:<region>:<accountId>:cert/<certificateId>`. You will use
this value when you attach the certificate to the AWS IoT policy.

### Register Your Edison and Attach to the Principal

In terms of AWS IoT, your Intel Edison device is a _thing_.  To start
registering your Edison with AWS IoT and attaching it to the
Certificate that was generated in the previous step.

1. Issue the following command to create the Thing:

   ``` bash
   $ aws iot create-thing --thing-name myEdison
   ```

2. If you have misplaced the `certificateArn` value, you can issue the
following command to locate it:

   ``` bash
   $ aws iot list-certificates
   ```

3. Issue the following command to attach the Thing to the Principal:

    ``` bash
    $ aws iot attach-thing-principal                   \
         --thing-name myEdison                          \
         --principal <certificate_arn>
    ```

### Create and Attach the AWS IoT Policy

Create a JSON policy document for the AWS IoT SDK.  For more
information about AWS IoT policies, see
[Authorization](http://docs.aws.amazon.com/iot/latest/developerguide/authorization.html)
in the service documentation.

The following policy allows all IoT actions and should be used for
development purposes only.

1. Copy the following text (CTRL+C):

   ``` json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": [
                   "iot:*"
               ],
               "Resource": [
                   "*"
               ]
           }
       ]
   }
   ``` 

2. Enter `vi policy.json`,  hit "a", and right-click to paste the text.
3. Press ESC and type in `:wq` to save and quit.
4. Create the IoT policy by issuing the following command:

   ``` bash
   $ aws iot create-policy                             \
        --policy-name EdisonPubSubToAnyTopic            \
        --policy-document file://policy.json
   ```
   
5. If you have misplaced the `certificateArn` value, you can issue the
following command to locate it:

   ``` bash
   $ aws iot list-certificates
   ```

6. Attach the policy to the certificate by issuing the following command:

   ``` bash
   $ aws iot attach-principal-policy                   \
        --principal <certificate_arn>                   \
        --policy-name EdisonPubSubToAnyTopic 
   ```

