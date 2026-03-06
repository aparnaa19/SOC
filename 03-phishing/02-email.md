# Email 

## Email Protocols

Several protocols work together when sending an email.

1.	SMTP (Simple Mail Transfer Protocol)
-	Purpose:
      - Sending emails
-	Default Port:
      - Port 25
-	Used by:
      - Mail servers to transfer emails.

2.	POP3 (Post Office Protocol)
-	Purpose:
      - Retrieves emails from the server.
-	Characteristics:
	    - Emails downloaded to one device
	    - Emails stored locally
	    - Cannot easily access from multiple devices
      - Server messages often deleted after download

3.	IMAP (Internet Message Access Protocol)
-	Purpose:
      - Retrieves emails from the server.
-	Characteristics:
      - Emails stored on server
      - Access from multiple devices
      - Messages synced across devices
      - Sent mail stored on server

----

## POP3 vs IMAP
| Feature | POP3 | IMAP |
|--------|------|------|
| Email storage | Local device | Mail server |
| Multi-device access | No | Yes |
| Sync support | No | Yes |
| Sent mail storage | Local | Server |

----

## How Email Travels (Step-by-Step)

Example:
Alexa sends an email to billy@johndoe.com

Step-by-Step Process
1.	Alexa composes an email and presses Send.
2.	Alexa's SMTP server determines where to send the email.
3.	SMTP queries DNS for the destination domain.
4.	DNS returns information about johndoe.com.
5.	The email is sent across the Internet.
6.	It travels through multiple SMTP servers.
7.	It reaches the destination SMTP server.
8.	The email is placed in the POP3/IMAP server mailbox.
9.	Billy opens his email client and retrieves the email.

----
## Email Address Structure

An email address has three parts:

“ Username@Domain “

1-	Username 
2-	Separator - @
3-	Domain name 

## Email Structure

An email consists of two main parts.

1. Email Header
  - Contains metadata about the message.
  - Example information:
        - Sender
        - Recipient
        - Servers used
        - Time stamps
        - Authentication results

2. Email Body
  - Contains the message content:
        - Plain text
        - HTML formatted content
        - Images
        - Links
        - Attachments

----

## Internet Message Format (IMF)
 - Emails follow a standard called:
 - This defines how	Headers, Body, Content types are structured inside email messages

## Basic Email Header Fields
- These fields are visible in most email clients
- These fields are the first things analysts inspect.
        - Field	Description
        - From	Sender email address
        - To	Recipient email address
        - Subject	Email topic
        - Date	Time the email was sent
- But viewing a Raw headers contain much more information than visible fields (raw header/ source code)

## Important Raw Header Fields for Analysis

1. X-Originating-IP
- IP address where the email originated
    - Example use:
          - Identify attacker location
          - Investigate suspicious hosts

2. SMTP Mail From / Header From
- Domain used to send the email
- Found inside:
      - Authentication-Results
- Used for:
      - Verifying sender legitimacy

3. Reply-To
- Specifies where replies are sent
- this is important because: 
      - When a user replies, it goes to the Reply-To address, not the sender.
      - Attackers often exploit this to redirect responses.

----

## Importance of Viewing Email HTML Source
- Security analysts can view the raw HTML behind emails.
- HTML reveals:
      - Hidden links
      - Embedded images
      - Tracking elements
      - Malicious scripts
      - email attachments
            - email attachement appears in email source with specific headers.
            - one such important attachment header is "Content-type"
            - Content type has the type of file that is attached
            - using this analyst can find any presence of executable files
     

