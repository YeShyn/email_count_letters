# email_count_letters
 counts the number of emails received from each sender.

### Description

If you have a lot of emails in your Gmail and want to clean up your inbox or unsubscribe from the most persistent senders, this script can help you count the number of emails from each sender. Using Python, this script connects to your mailbox, retrieves all emails for a specified period, and counts the number of emails received from each sender.

### Features

- Connects to Gmail via IMAP using SSL.
- Searches and processes all emails within a specified period.
- Counts the number of emails from each sender.
- Handles errors during connection and email retrieval.

### Requirements

- Python 3.x
- Installed libraries:
    - `imaplib`
    - `ssl`
    - `email`
    - `collections`
    - `datetime`

### Usage

1. Edit the script to include your Gmail credentials:
```python
IMAP_SERVER = 'imap.gmail.com' 
EMAIL_ACCOUNT = 'youremail@gmail.com' 
PASSWORD = 'your_app_password'
```
2. Run the script in Jupyter Notebook.

## Jupyter Notebook Code

Make sure to run each cell sequentially in your Jupyter Notebook:
```python
import imaplib
import email
from email.header import decode_header
from collections import Counter
from datetime import datetime, timedelta
import ssl

# Credentials
IMAP_SERVER = 'imap.gmail.com'
EMAIL_ACCOUNT = 'youremail@gmail.com'  # Replace with your email address
PASSWORD = 'your_app_password'  # Your Google App password (https://myaccount.google.com/apppasswords)

def connect_to_mail():
    context = ssl.create_default_context()
    mail = imaplib.IMAP4_SSL(IMAP_SERVER, ssl_context=context)
    mail.login(EMAIL_ACCOUNT, PASSWORD)
    mail.select('inbox')
    return mail

def process_email(email_id, mail):
    try:
        result, msg_data = mail.fetch(email_id, '(RFC822)')
        if result == 'OK':
            raw_email = msg_data[0][1]
            msg = email.message_from_bytes(raw_email)
            
            from_header = msg['From']
            decoded_header = decode_header(from_header)
            for decoded_string, charset in decoded_header:
                if isinstance(decoded_string, bytes):
                    charset = charset or 'utf-8'
                    try:
                        decoded_string = decoded_string.decode(charset)
                    except LookupError:
                        decoded_string = decoded_string.decode('utf-8', errors='ignore')
                sender_email = email.utils.parseaddr(decoded_string)[1]
                if sender_email:
                    return sender_email
    except imaplib.IMAP4.abort as e:
        print(f"Error fetching email {email_id}: {e}")
    except Exception as e:
        print(f"Unexpected error fetching email {email_id}: {e}")
    return None

def fetch_emails_since(mail, date_since):
    result, data = mail.search(None, f'SINCE {date_since}')
    if result == 'OK':
        return data[0].split()
    else:
        print('Error searching emails')
        return []

# Connect to the server
mail = connect_to_mail()

# Define the date to limit the email search (e.g., emails from the last month)
date_since = (datetime.now() - timedelta(days=30)).strftime("%d-%b-%Y")

# Search for emails received after a certain date
email_ids = fetch_emails_since(mail, date_since)

senders = []

# Process each email
for email_id in email_ids:
    sender_email = process_email(email_id, mail)
    if sender_email:
        senders.append(sender_email)

# Count the number of emails from each sender
sender_counts = Counter(senders)

# Display the results
for sender, count in sender_counts.items():
    print(f'{sender}: {count} emails')

# Close the connection
mail.logout()

```