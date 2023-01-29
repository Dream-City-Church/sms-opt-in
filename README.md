# SMS Opt-In with Ministry Platform
### In order for an organization to comply with SMS regulations and requirements from both government and wireless carriers, they must have an opt-in process for their SMS campaigns.

### Dream City Church has been developing an opt-in process within Ministry Platform so that we can not only respect communication preferences, but also meet requirements for shortcodes and 800 numbers.

### We set out to make this process as available to as many Ministry Platform churches as possible, meaning we tried to keep the core functionality as "stock" as possible. We will be adding additional functionality with database customizations and Azure Logic Apps to expand the functionality and reporting capabilities, but these should be optional.


## Texting Setup in Ministry Platform
Please review the Ministry Platform Knowledge Base article on Texting: https://www.ministryplatform.com/kb/ministryplatform/communications/messages/texting
* Set up Outbound SMS Numbers
* Set up Inbound Messages
* Note the section on *Opting Out and Opting Back In*. Ministry Platform will automatically set a Contact's "Do Not Text" field based on if an inbound message contains one of the opt-in or opt-out keywords.

You will also want to create a template message for the opt-in/double opt-in request:
> DCC: Thanks for connecting with us! Reply 'Yes' to receive church news and announcements. STOP to cancel. Msg frequency varies. Standard msg&data rates apply.

## Process Decision Flow
There's 6 different scenarios that need to be considered when handling a mobile phone number and whether to send an opt-in message:

1. New Contact with new mobile phone number
2. New Contact with mobile phone number matching existing contacts that are only opt-in
3. New Contact with mobile phone number matching existing contacts that are already opt-out
4. Updated Contact with new mobile phone number
5. Updated Cotnact with mobile phone number matching existing contacts that are only opt-in
6. Updated contact with mobile phone number matching existing contact that is already opt-out

The default is "Do_Not_Text=0", meaning by default someone is "subscribed" to SMS communication.

With that in mind, we will create a process that will set "Do_Not_Text=1" on submit for Contacts that are in scenarios 1, 3, 4, and 6 above (we don't need to do anything for Contacts that are in scenarios 2 and 5 as they are by default "opted-in" and we're assuming someone with that number has already opted in)

We will also send an opt-in message to contacts in scenarios 1, 5, and 6. What about scenario 3? In the event that several new Contacts are creatd with the same mobile phone number (entering a new family, for example, it's common that a parent's number is entered on their kids Cotnact records) we don't want to send an opt-in message several times at once. That is less likely to occur when a contact's mobile phone is updated and matches an existing "do not text", and we would like to give the opportunity to re-subscribe. You can of course customize the processes if you would like to or not like to send opt-in messages to your people.