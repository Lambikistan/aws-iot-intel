# Getting Started with AWS IoT on Intel Edison

![Intel Edison + AWS IoT](https://cloud.githubusercontent.com/assets/2881361/10375573/439555c4-6dc7-11e5-8eb0-75b9f1506f30.png)

## Prepare Your Intel Edison

Get started with your Intel Edison by updating to the latest firmware
and setting up a serial terminal. You can find the instructions here:

[Get started with Intel Edison technology](https://software.intel.com/en-us/iot/library/edison-getting-started)

## Prepare Your Intel Edison for AWS IoT

Before continuing, it is suggested that you read the AWS IoT
[Quickstart](http://docs.aws.amazon.com/iot/latest/developerguide/iot-quickstart.html).
If you are already familiar with AWS IoT, please continue to get AWS
interoperability functioning on your Edison device.

### Install the AWS CLI

The AWS CLI is used to interoperate with Amazon Web Services.  The
installation of Groff and a different version of less is required to
view the help documentation.

At this point, we assume you have internet access from the device to
download and install these packages.

### Install the Python Package Manager (pip)

   ``` bash
   $ curl https://bootstrap.pypa.io/ez_setup.py -o - | python
   $ easy_install pip
   ```

### Install the AWS CLI

   Install the AWS CLI by issuing the following command:

   ``` bash
   $ pip install awscli
   ``` 

### Install Dependencies

To view help files using `aws iot help`, the Groff and a non-BusyBox
version of less packages are required.

#### Groff Installation

Execute the following commands to install Groff.

   ``` bash
   $ wget http://ftp.gnu.org/gnu/groff/groff-1.22.3.tar.gz
   $ tar -zxvf groff-1.22.3.tar.gz
   $ cd groff-1.22.3
   $ ./configure
   $ make
   $ make install
   $ export PATH=$PATH:/usr/local/bin/
   $ cd ~
   ``` 
#### Less Installation

First, rename the old version of less:

   ``` bash
   $ mv /usr/bin/less /usr/bin/less-OLD
   ``` 

Next, install the new version of less:

   ``` bash
   $ wget http://www.greenwoodsoftware.com/less/less-458.zip
   $ unzip less-458.zip
   $ cd less-458
   $ chmod 777 *
   $ ./configure
   $ make
   $ make install
   $ cd ~
   ```

To make sure everything is installed correctly, run the IoT help file:

   ``` bash
   $ aws iot help
   ``` 

## Setting up Your Edison as an AWS IoT Thing

### Get AWS Credentials

The AWS CLI is now installed. Create a new IAM user and get API
credentials from the AWS Management Console by following
[Getting Set Up with the AWS Command Line Interface](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-set-up.html#cli-signup). After
you have an access ID and key, configure the AWS CLI credentials by
issuing the following command:

   ``` bash 
   $ aws configure 
   ```

**NOTE:** For the default region you must enter us-east-1 in order to
  configure AWS IoT. The default output format can be left as JSON.

In order to get permission to download the AWS IoT tools, attach the
administrator account policy to the user. In the IAM console, in the
Users panel, select the user you created, attach the policy, and then
select the administrator account.

### Register Your Edison
In terms of AWS IoT, your Intel Edison device is a _Thing_.  Begin by registering your Edison with AWS IoT by issuing the following command:

   ``` bash
   $ aws iot create-thing --thing-name myEdison
   ```

### Generate Certificates

1. Create a folder where your certificates will be stored:

   ``` bash
   $ cd ~
   $ mkdir aws_certs
   $ chmod 700 aws_certs
   $ cd aws_certs
   ```

2. A client certificate must be generated to authenticate with AWS IoT
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

### Create and Attach the AWS IoT Policy

Create a JSON policy document for the AWS IoT SDK.  For more
information about AWS IoT policies, see
[Authorization](http://docs.aws.amazon.com/iot/latest/developerguide/authorization.html)
in the service documentation.

The following policy allows all IoT actions and should be used for
development purposes only.

1. Copy the following text (CTRL-C):

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

### Attach the Policy to Your Certificate

1. Create the IoT policy by issuing the following command:

   ``` bash
   $ aws iot create-policy                             \
        --policy-name PubSubToAnyTopic                  \
        --policy-document file://policy.json
   ``` 

2. If you have misplaced your certificate ID, you can issue the
following command to locate it:

   ``` bash
   $ aws iot list-certificates
   ```

3. Attach the policy to the certificate by issuing the following command:

   ``` bash
   $ aws iot attach-principal-policy                   \
        --principal-arn <principal arn>                 \
        --policy-name "PubSubToAnyTopic" 
   ``` 
