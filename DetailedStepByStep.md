
**Log in**

Navigate to https://cloud.elastic.co/ and log in or sign up for a free (as in no credit card needed!) trial

![ScreenShot2017-10-16at9.48.16AM](https://user-images.githubusercontent.com/25182304/31694577-33ad18f6-b373-11e7-80ef-1f6e61b37c2d.png)

**Create a cluster**

You can accept the defaults, there is one option in there to ask for high availability across two data centers, this time I just took the default

![ScreenShot2017-10-16at9.48.38AM](https://user-images.githubusercontent.com/25182304/31694578-33bd5946-b373-11e7-8fb4-34f33657a979.png)

**Don't Panic**

I know it mentions $120 per month, that is if you decide to keep it.  They will not bill you automatically, in fact they will ask you at the end if you want to subscribe and will even store your data for a bit after the 14 days

![ScreenShot2017-10-16at9.49.37AM](https://user-images.githubusercontent.com/25182304/31694579-33cee788-b373-11e7-89f8-05df1d7e1b8e.png)

**Store your password in a safe place**

![ScreenShot2017-10-16at9.50.20AM](https://user-images.githubusercontent.com/25182304/31694580-33d9f3d0-b373-11e7-9e4a-213e5747e944.png)

**Save this URL**

Take note of your endpoint, this is where you will send data to

![ScreenShot2017-10-16at9.50.59AM](https://user-images.githubusercontent.com/25182304/31694581-33e88ac6-b373-11e7-8704-46127606fc06.png)

**Save your Kibana URL**

The Kibana URL is under Configuration

![ScreenShot2017-10-16at9.55.13AM](https://user-images.githubusercontent.com/25182304/31694582-33fabcd2-b373-11e7-88f1-af4724891d7f.png)

**Connect to Kibana**

Log in as user **elastic** with the password you wrote down above

![ScreenShot2017-10-16at9.59.59AM](https://user-images.githubusercontent.com/25182304/31694583-340a21ea-b373-11e7-947c-10cbea71790f.png)

**Ignore the index warning**

When you send data in you will be able to choose a default index, don't worry about this for now.  Click on Management in the left navigation, and then on Management in the breadcrumbs on the right

![ScreenShot2017-10-16at10.00.25AM](https://user-images.githubusercontent.com/25182304/31694584-3418e392-b373-11e7-95c5-8146ddceac13.png)

**Closeup of the right side breadcrumb**

![ScreenShot2017-10-16at10.01.35AM](https://user-images.githubusercontent.com/25182304/31694585-3428af66-b373-11e7-9dbf-1016fab56941.png)

**See the docs on authentication**

There are docs for setting up authentication (try not to make every account a superuser!) at http://bit.ly/2zuiaRz .  The Logstash account does not need very many privileges, but does need to be able to create indexes and write to them.

![ScreenShot2017-10-16at10.15.21AM](https://user-images.githubusercontent.com/25182304/31694586-34387edc-b373-11e7-8036-91d029631b1c.png)

**Create a Role with X-Pack**

Back in Kibana, under Management, Management click on Roles

![ScreenShot2017-10-16at10.19.29AM](https://user-images.githubusercontent.com/25182304/31694588-345c8c1e-b373-11e7-93ac-b0f0580a5e0e.png)

**Create a logstash_writer role**

In the **Name** and **Indices** fields you type in the role name and either a specific index name, or a wildcard.  I use this role for multiple Logstash streams, so I use a wildcard (see below).  Check the box for the **manage_index_templates** Cluster Privilege, and then click in the Index Privileges box and choose the privileges below (which match the docs we looked at above)

![ScreenShot2017-10-16at10.19.09AM](https://user-images.githubusercontent.com/25182304/31694587-344b0c14-b373-11e7-816c-6b6b7a92df20.png)



**Add a user**

Add a user with the role created above.  I use the suffix **_agent** to indicate that a process running on an external machine is using this account.  Store the username and password with the rest of your notes.

![ScreenShot2017-10-16at10.21.18AM](https://user-images.githubusercontent.com/25182304/31694589-346d6d0e-b373-11e7-8c87-4004669ad2c8.png)

**Get some data**

I like to use known sample data that is pretty common.  Generally I use either syslog data or Apache HTTP Server logs.  There is a sample data set and tutorial at http://bit.ly/2ijecXK .  In fact, I recommend going through the tutorial later to learn about using Kibana templates and geoip (it is a great tutorial).  For now, just grab the Appache HTTP Server logs and Logstash config files.

![ScreenShot2017-10-16at10.27.11AM](https://user-images.githubusercontent.com/25182304/31694591-348b554e-b373-11e7-92c8-6a4cfa4b15c4.png)

b

![ScreenShot2017-10-16at10.29.05AM](https://user-images.githubusercontent.com/25182304/31694592-349a2f74-b373-11e7-8271-89c9d60cb753.png)

c

![ScreenShot2017-10-16at10.30.25AM](https://user-images.githubusercontent.com/25182304/31694595-34bd3ae6-b373-11e7-9f4e-b7701d64b484.png)

b

![ScreenShot2017-10-16at10.29.36AM](https://user-images.githubusercontent.com/25182304/31694596-34ccfed6-b373-11e7-8154-124c0b634258.png)

b

![ScreenShot2017-10-16at10.39.22AM](https://user-images.githubusercontent.com/25182304/31694597-34de3f66-b373-11e7-9c14-a4de7c29522a.png)

b

![ScreenShot2017-10-16at10.39.44AM](https://user-images.githubusercontent.com/25182304/31694598-34ee1e04-b373-11e7-868e-5987820fd391.png)

b

![ScreenShot2017-10-16at11.19.39AM](https://user-images.githubusercontent.com/25182304/31694602-354a1466-b373-11e7-88c7-3d2f98018f3a.png)

b

![ScreenShot2017-10-16at11.19.11AM](https://user-images.githubusercontent.com/25182304/31694601-3538f3b6-b373-11e7-945d-5f5d1d71f331.png)

b

![ScreenShot2017-10-16at11.20.16AM](https://user-images.githubusercontent.com/25182304/31694603-3560d6b0-b373-11e7-84c6-64f58d529551.png)

b

![ScreenShot2017-10-16at11.25.31AM](https://user-images.githubusercontent.com/25182304/31694604-356e518c-b373-11e7-80e0-23c3a61d6637.png)

b

![ScreenShot2017-10-16at11.27.01AM](https://user-images.githubusercontent.com/25182304/31694605-357a7bc4-b373-11e7-923c-07cfcfae9c7f.png)

b

![ScreenShot2017-10-16at11.37.27AM](https://user-images.githubusercontent.com/25182304/31694606-3586805e-b373-11e7-916c-5dc3556adc72.png)

b

![ScreenShot2017-10-16at11.37.43AM](https://user-images.githubusercontent.com/25182304/31694607-3598b9b8-b373-11e7-915e-d188b7d25211.png)

b

![ScreenShot2017-10-16at11.37.53AM](https://user-images.githubusercontent.com/25182304/31694608-35a8bc1e-b373-11e7-8f31-83dcd90c33dd.png)

a

a![ScreenShot2017-10-16at11.38.34AM](https://user-images.githubusercontent.com/25182304/31694609-35c733f6-b373-11e7-902d-d9271269770c.png)

![ScreenShot2017-10-16at11.48.25AM](https://user-images.githubusercontent.com/25182304/31694610-35e2cd64-b373-11e7-92cc-e17ce945af74.png)

a

![ScreenShot2017-10-16at11.48.39AM](https://user-images.githubusercontent.com/25182304/31694611-3600c972-b373-11e7-94f7-ca77244cd51d.png)

a

![ScreenShot2017-10-16at11.48.52AM](https://user-images.githubusercontent.com/25182304/31694612-3610b846-b373-11e7-99bc-ce720483bc62.png)

a

![ScreenShot2017-10-16at11.49.42AM](https://user-images.githubusercontent.com/25182304/31694614-36387f5c-b373-11e7-8ccc-ea4dbb2a382f.png)



a

![ScreenShot2017-10-16at11.50.55AM](https://user-images.githubusercontent.com/25182304/31694615-364a9c8c-b373-11e7-88d3-7fdd3ee05fc4.png)

a

![ScreenShot2017-10-16at11.51.08AM](https://user-images.githubusercontent.com/25182304/31694616-36654442-b373-11e7-9e8f-c9be22a6825e.png)
