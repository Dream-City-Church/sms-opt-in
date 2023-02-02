# SMS Opt-In with Ministry Platform
In order for an organization to comply with SMS regulations and requirements from both government and wireless carriers, they must have an opt-in process for their SMS campaigns.

Dream City Church has been developing an opt-in process within Ministry Platform so that we can not only respect communication preferences, but also meet requirements.

We set out to make this process as available to as many Ministry Platform churches as possible, meaning we tried to keep the core functionality as "stock" as possible. Over time, DCC will be adding additional functionality with database customizations and Azure Logic Apps to expand the functionality and reporting capabilities, which we'll share at the end of this document.


## Texting Setup in Ministry Platform
Please review the Ministry Platform Knowledge Base article on Texting: https://www.ministryplatform.com/kb/ministryplatform/communications/messages/texting
* Set up Outbound SMS Numbers
* Set up Inbound Messages
* Note the section on *Opting Out and Opting Back In*. Ministry Platform will automatically set Contact's "Do Not Text" field based on if an inbound message contains one of the opt-in or opt-out keywords.

You will also want to create a template message in MP for the opt-in/double opt-in request:
> DCC: Thanks for connecting with us! Reply 'Yes' to receive church news and announcements. STOP to cancel. Msg frequency varies. Standard msg&data rates apply.

You may also want to set up Messaging Service Opt-Out Management on your Twilio numbers. This will allow you to customize keywords and the reply messages when an Opt-Out, Opt-In, or Help message is received.

# Process Decision Flow
There's 6 different scenarios that need to be considered when handling a mobile phone number and whether to send an opt-in message:

1. New Contact with new mobile phone number
2. New Contact with mobile phone number matching existing contacts that are only opt-in
3. New Contact with mobile phone number matching existing contact that is already opt-out
4. Updated Contact with new mobile phone number
5. Updated Contact with mobile phone number matching existing contacts that are only opt-in
6. Updated contact with mobile phone number matching existing contact that is already opt-out

The default is "Do_Not_Text=0", meaning by default someone is "subscribed" to SMS communication.

With that in mind, we will create processes that will set "Do_Not_Text=1" on submit for Contacts that are in scenarios 1, 3, 4, and 6, and "Do_Not_Text=0" for contacts in scenario 5 (we don't need to do anything for Contacts that are in scenarios 2 as they are by default "opted-in" and we're assuming someone with that number has already opted in)

We will also need to send an opt-in message to contacts in scenarios 1 and 4 as those are "new" numbers that we want to try and subscribe.

# The 5 Processes in MP
Create each of the 5 Processes below in Ministry Platform. You may need to modify the Dependent Conditions in order to meet the specific goals of your church.
## #1 - New Contact New Number
### Process: General
Create your process in MP with the below configuration:
* Process Name: _Your Choice_
* Process Manger: _Your Choice_
* Active: Yes
* Description: Send an opt-in message to new contacts with mobile numbers that do not match any other contacts.
* On Submit: Do_Not_Text=1
* On Complete:
* Trigger Fields: 
* Dependent Condition: 
> Mobile_Phone IS NOT NULL  
AND NOT EXISTS(SELECT * FROM Contacts C  
	WHERE C.Contact_ID <> Contacts.Contact_ID  
	AND C.Mobile_Phone = Contacts.Mobile_Phone)  
AND EXISTS(SELECT * FROM Contacts C   
	LEFT JOIN Participants P ON P.Contact_ID = C.Contact_ID    
	WHERE C.Contact_ID = Contacts.Contact_ID   
	AND P.Participant_Type_ID NOT IN (_1, 2, 3, etc._))  
* Trigger On Create: Yes
* Trigger On Update: No
* Table Name: Contacts

### Process: Steps
Create your process step in MP with the below configuration (blank fields skipped for brevity):
* Step Name: Send SMS Opt In
* Step Type: Send a Text
* Escalation Only: No
* Order: 0
* Supervisor User: No
* Text Template: _The SMS template you created earlier_
* Text To Contact Field: Contacts.Contact_ID

### Explanation

Let's break down the SQL in the Dependent Condition so that you can understand what it is looking for and how you can modify it for your church.
>Mobile_Phone IS NOT NULL

The mobile phone field has a value.
>AND NOT EXISTS(SELECT * FROM Contacts C  
WHERE C.Contact_ID <> Contacts.Contact_ID  
AND C.Mobile_Phone = Contacts.Mobile_Phone)

There does not exist any contact with a different Contact ID from the current contact and that has a matching phone number to the current contact.

>AND EXISTS(SELECT * FROM Contacts C   
LEFT JOIN Participants P ON P.Contact_ID = C.Contact_ID   
WHERE C.Contact_ID = Contacts.Contact_ID  
AND P.Participant_Type_ID NOT IN (_1,2,3,etc...._))

There's a couple different things happening in this block. Because the Dependent Conditions field does not allow table lookup convention, we have to use a select statement to query related tables on the current record.

>WHERE C.Contact_ID = Contacts.Contact_ID

simply means we're only interested in the current contact.

>AND P.Participant_Type_ID NOT IN (_1,2,3,etc...._)

We don't want to send an opt-in SMS to contacts that are in a few specific types (companies, deceased, etc.). You could also use the inverse and use Participant_Type_ID IN (1,2,3) if you want to be more restrictive.

This would be a good place to add any other filters for things like member status, congregations, accounting companies, etc.

## #2 - New Contact Matches Opted-Out Number
### Process: General
Create your process in MP with the below configuration:
* Process Name: _Your Choice_
* Process Manger: _Your Choice_
* Active: Yes
* Description: Sets a new contact's Do Not Text to true if another contact with the same mobile number is already set to true.
* On Submit: Do_Not_Text=1
* On Complete:
* Trigger Fields: 
* Dependent Condition: 
> Mobile_Phone IS NOT NULL   
AND EXISTS(SELECT * FROM Contacts C  
WHERE C.Contact_ID <> Contacts.Contact_ID  
AND C.Mobile_Phone = Contacts.Mobile_Phone  
AND C.Do_Not_Text = 1)
* Trigger On Create: Yes
* Trigger On Update: No
* Table Name: Contacts

### Process: Steps
There are no 'Steps' associated with this process.

### Explanation

If a contact with a different Contact ID, a matching mobile number, and is Do_Not_Text=1, then we'll set this new contact to also be Do_Not_Text=1 as that mobile number has not opted-in.

## #3 - Updated Contact New Number
### Process: General
Create your process in MP with the below configuration:
* Process Name: _Your Choice_
* Process Manger: _Your Choice_
* Active: Yes
* Description: Send an opt-in message to updated contacts with mobile numbers that do not match any other contacts.
* On Submit: Do_Not_Text=1
* On Complete:
* Trigger Fields: Mobile_Phone
* Dependent Condition: 
> Mobile_Phone IS NOT NULL  
AND NOT EXISTS(SELECT * FROM Contacts C  
	WHERE C.Contact_ID <> Contacts.Contact_ID  
	AND C.Mobile_Phone = Contacts.Mobile_Phone)  
AND EXISTS(SELECT * FROM Contacts C   
	LEFT JOIN Participants P ON P.Contact_ID = C.Contact_ID    
	WHERE C.Contact_ID = Contacts.Contact_ID   
	AND P.Participant_Type_ID NOT IN (_1, 2, 3, etc._))  
* Trigger On Create: No
* Trigger On Update: Yes
* Table Name: Contacts

### Process: Steps
Create your process step in MP with the below configuration (blank fields skipped for brevity):
* Step Name: Send SMS Opt In
* Step Type: Send a Text
* Escalation Only: No
* Order: 0
* Supervisor User: No
* Text Template: _The SMS template you created earlier_
* Text To Contact Field: Contacts.Contact_ID

### Explanation

Very similar to our first process, except we are only firing on an update and only if the Mobile_Phone field has been modified.

## #4 - Updated Contact Matches Opted-Out Number
### Process: General
Create your process in MP with the below configuration:
* Process Name: _Your Choice_
* Process Manger: _Your Choice_
* Active: Yes
* Description: Sets an updated contact's Do Not Text to true if another contact with the same mobile number is already set to true.
* On Submit: Do_Not_Text=1
* On Complete:
* Trigger Fields: Mobile_Phone
* Dependent Condition: 
> Mobile_Phone IS NOT NULL   
AND EXISTS(SELECT * FROM Contacts C  
WHERE C.Contact_ID <> Contacts.Contact_ID  
AND C.Mobile_Phone = Contacts.Mobile_Phone  
AND C.Do_Not_Text = 1)
* Trigger On Create: No
* Trigger On Update: Yes
* Table Name: Contacts

### Process: Steps
There are no 'Steps' associated with this process.

### Explanation

Again, very similar to the process for new contacts, but scoped to just when an existing contact's mobile phone field is modified.

## #5 - Updated Contact Matches Opted-Out Number
### Process: General
Create your process in MP with the below configuration:
* Process Name: _Your Choice_
* Process Manger: _Your Choice_
* Active: Yes
* Description: Sets an updated contact's Do Not Text to false if another contact with the same mobile number is already set to false.
* On Submit: Do_Not_Text=0
* On Complete:
* Trigger Fields: Mobile_Phone
* Dependent Condition: 
> Mobile_Phone IS NOT NULL  
AND Do_Not_Text=1  
AND EXISTS(SELECT * FROM Contacts C  
WHERE C.Contact_ID <> Contacts.Contact_ID  
AND C.Mobile_Phone = Contacts.Mobile_Phone  
AND C.Do_Not_Text = 0)  
AND NOT EXISTS(SELECT * FROM Contacts C  
WHERE C.Contact_ID <> Contacts.Contact_ID  
AND C.Mobile_Phone = Contacts.Mobile_Phone  
AND C.Do_Not_Text = 1)
* Trigger On Create: No
* Trigger On Update: Yes
* Table Name: Contacts

### Process: Steps
There are no 'Steps' associated with this process.

### Explanation
This last process addresses a rare occurence where a contact who was previously do_not_text=1 changes their phone number to one that matches on other contacts that only have do_not_text=0. Because that mobile number has already opted-in, we will match that preference to this modified contact.


# Database Customization & API Integration
We have not implemented any customizations yet.
- - -

Idea: New database table to track the opt in status and facts of each mobile phone number ever entered into Ministry Platform

Table: Mobile_Phone_Numbers 

Columns:  
* Mobile_Phone_ID  
* Mobile_Phone (Unique)  
* Do_Not_Text  
* _Created_Date  
* _Last_Opt_In_Attempt_Date  
* _Opt_In_Received_Date  
* _Opt_Out_Received_Date  

Add "Mobile_Phone_ID" column to Contacts table.

Use API/Azure Logic Apps or stored procedures to update records in the Mobile_Phone_Numbers table and update related Contacts. By tracking "_Last_Opt_In_Attempt_Date" we can re-send invites during activity touch points (e.g. registered for an event) without "spamming" the recipient.