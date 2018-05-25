# remote-authorize
An Adobe PhoneGap app with two-factor authorization of checks for QuickBooks Bill Pay

![PhoneGap Client screenshot](https://cloud.githubusercontent.com/assets/15971213/12862093/277c63c8-cc1d-11e5-8f4a-688328e04724.png)

### Accounting Controls

In the world of accounting there's a concept of "Best Practices" with respect to the separation of duties.  Specifically, a critical practice is the separation of these *particular two activities*:  1) creating a check and 2) signing that check.  This control helps to minimize theft and is seen as the cornerstone of controlling assets.  These types of controls have been in place for some time now (think: a century or more).

### Today's Accounting

Fast-forward to today's state-of-the-art technology and we have a number of advancements:  cellphones, online bill pay, two-factor authentication, public-key cryptography and virtual signatures.  And yet, there doesn't appear to be a solution so that the Accounts Payable clerk may submit an online Bill Pay check, easily notify the signator(s) who may then go online and authorize that check.  In this scenario with both QuickBooks as the accounting system and Wells Fargo as the custodial bank, this would require a signator to possess the online credentials to the checking account.  In the realm of accounting controls, this is a bad idea.  You just need a signator (a Vice President of the company, for example) to sign the check and nothing more.

### Background

QuickBooks has a feature called Bill Pay which allows a check to be submitted to be paid.  During the setup of this feature it may be configured to be submitted but not yet authorized to go out.  In the case of Wells Fargo online banking, the owner of the checking account may sign into the website and authorize checks from this queue.  This probably works well for smaller companies who authorize few checks but it's a pain for larger companies.  And this is especially a problem when the owner is usually on the road.  Even so, there may be times when the other signators for that checking account are *also* away from the office.  It's not realistic to wait until one of them returns.

### Work-arounds

Usually what happens in smaller companies in this scenario is that the owner pre-signs a short stack of checks or leaves a stamp which allows a reproduction of their signature to be placed onto newly-created checks.  You can see how this might open the door to theft in some circumstances.

### Assumptions

Here, I make some assumptions about the setup of the company in question:

* Some form of multi-user QuickBooks is installed locally at the office and that there is a free license slot available for an application to run
* Each of the signators have a smartphone that has an iOS, Android or Microsoft Phone operating system and which has either wi-fi or data enabled
* The Accounts Payable clerk has the ability to send a text message to the collection of signators
* That a Wells Fargo or similar checking account is being used which supports QuickBooks's online Bill Pay feature

### Requirements

In order to make everyone happy, a good solution would need to minimally do the following:

* Enable the original accounting control of separating those-who-write-checks from those-who-authorize-them
* Employ some form of two-factor authentication for the signators
* Include an auditing feature to determine who authorized the check, who initiated the check and all status to include timestamps
* Efficiently notify the signator(s) and if necessary arbitrate which of them should sign
* Concisely display the pertinent check information to the signator

### Solution

In this scenario it looks like the following will need to be built.

#### 1. QuickBooks application for the AP clerk

The QuickBooks application will be authorized to run by the I.T. administrator and then will be called up as checks are submitted online via the Bill Pay feature.

The QuickBooks application itself will have no knowledge of the online banking account's credentials nor will this appear in any form of server/client communication.

In the absence of this piece a work-around may be initially used instead: The server application may host an HTML form which authenticates the AP clerk and then allows them to enter the pertinent details of the check which has been submitted to Bill Pay.

#### 2. Server application

This will be a Node.js REST-based website.  It will host the audit log, the signator authorization table, the queue of checks to be signed.  Additionally, it will be the endpoints for both the QB and PhoneGap app communication and will be responsible for communicating to the online banking site to authorize each check in question.  This will be the only piece which knows the online banking account's credentials.

There doesn't appear to be an easy method of **not** using a server since it requires an audit log minimally for the solution to be trusted.  There could be some means of removing this piece but I wouldn't advise it.

#### 3. Client application for the signators

This will be an Adobe PhoneGap app and is the piece which is represented by this Github repository.  Each signator will have this installed on their phone and an initialization step will register their phone's UUID for half of the two-factor authentication requirement as well as create a new PIN number to be used for the second half.

The client application will have no knowledge of the online banking account's credentials nor will this appear in any form of server/client communication.

There doesn't appear to be an easy method of **not** using a phone app since we're trying to allow the signators the ability to 1) be away from the office and 2) be away from a computer.

### Installing

I assume that you have Node.js and PhoneGap loaded already on your workstation and that you're familiar with it.  Also, it would be good to have the PhoneGap Developer app for local testing and development. I'd suggest starting by installing the server-side of this code from the related repository.  Then when you're ready for the client part, do another git clone of this repository.

    git clone https://github.com/OutsourcedGuru/remote-authorize.git

Next, change into the directory created.

    cd remote-authorize

You should edit the config file to identify where your server and client are located and what ports they use.

    vi www/js/config.js

In development, the server is on port 3001 and the client is on 3000.

I assume that you've installed the server already.  Here's the general ordering of events after you've cloned both repositories and edited the appropriate settings.

1. Start the mongo database daemon
2. Optionally, start a Mongo shell session to see what's happening in the database
3. In another Terminal session, start the server with either `npm start` or for more information in the log, `set DEBUG=remote-authorize-server:* & npm start`.
4. Using your browser, visit `http://localhost:3001` to log into the server as an Accounts Payable clerk and to create a check-signing request
5. Using your browser, visit `http://localhost:3000` to see what the smartphone screen looks like for a signator (someone authorized to sign checks)
6. Optionally, use the PhoneGap app on your smartphone.  In this case you'll want to change the `strServer` address in `www/js/config.js` in the client code to the IP address of your server.  Direct the PhoneGap app to go to this address on port 3000. 
The server will detect that the phone's `uuid` hasn't been seen yet and go through an authorization session. 

|Donate||Cryptocurrency|
|:-----:|---|:--------:|
| ![eth-receive](https://user-images.githubusercontent.com/15971213/40564950-932d4d10-601f-11e8-90f0-459f8b32f01c.png) || ![btc-receive](https://user-images.githubusercontent.com/15971213/40564971-a2826002-601f-11e8-8d5e-eeb35ab53300.png) |
|Ethereum||Bitcoin|
