# Sendemail

```
sendEmail -t to@domain.com -f from@megacorpone.com -s <ip smtp> -u "Important subject" -a /tmp/malware-letter.pdf
Reading message body from STDIN because the '-m' option was not used.

If you are manually typing in a message:
  - First line must be received within 60 seconds.
  - End manual input with a CTRL-D on its own line.

Try Harder!
```

# Scripts to send an email

## Python

send_email.py
```
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
import argparse
import getpass

def send_email(subject, body, to_email, from_email, from_password):
    # SMTP server configuration
    smtp_server = "smtp.gmail.com"
    smtp_port = 587

    # Create the email message
    msg = MIMEMultipart()
    msg['From'] = from_email
    msg['To'] = to_email
    msg['Subject'] = subject

    # Attach the email body to the message
    msg.attach(MIMEText(body, 'plain'))

    try:
        # Connect to the SMTP server
        server = smtplib.SMTP(smtp_server, smtp_port)
        server.starttls()  # Upgrade the connection to a secure encrypted SSL/TTLS connection
        server.login(from_email, from_password)

        # Send the email
        server.sendmail(from_email, to_email, msg.as_string())
        print(f"Email sent successfully to {to_email}")

    except Exception as e:
        print(f"Failed to send email to {to_email}: {e}")

    finally:
        server.quit()

def read_email_list(file_path):
    with open(file_path, 'r') as file:
        email_list = [line.strip() for line in file if line.strip()]
    return email_list

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description='Send an email from the command line.')
    parser.add_argument('subject', help='The subject of the email')
    parser.add_argument('body', help='The body of the email')
    group = parser.add_mutually_exclusive_group(required=True)
    group.add_argument('--to_email', help='The recipient email address')
    group.add_argument('--email_list', help='Path to a file containing a list of recipient email addresses')
    parser.add_argument('--from_email', help='The sender email address')
    parser.add_argument('--from_password', help='The sender email password')
    args = parser.parse_args()

    if not args.from_email:
        args.from_email = input("Enter your email: ")
    if not args.from_password:
        args.from_password = getpass.getpass("Enter your password: ")

    if args.to_email:
        send_email(args.subject, args.body, args.to_email, args.from_email, args.from_password)
    elif args.email_list:
        email_list = read_email_list(args.email_list)
        for email in email_list:
            send_email(args.subject, args.body, email, args.from_email, args.from_password)
```

Command Options: 

1. Send an email to a single recipient: 

- python send_email.py "Test Subject" "This is the body of the email" --to_email recipient@example.com --from_email your-email@gmail.com --from_password your-password

2. Send the email to multiple recipients from a list: 

- python send_email.py "Test Subject" "This is the body of the email" --email_list email-list.txt --from_email your-email@gmail.com --from_password your-password

## Golang

send-email.go
```
package main

import (
	"bufio"
	"bytes"
	"flag"
	"fmt"
	"log"
	"net/smtp"
	"os"
	"strings"
)

func sendEmail(smtpHost string, smtpPort string, from string, password string, to []string, subject string, body string) error {
	auth := smtp.PlainAuth("", from, password, smtpHost)

	msg := bytes.Buffer{}
	msg.WriteString("From: " + from + "\n")
	msg.WriteString("To: " + strings.Join(to, ",") + "\n")
	msg.WriteString("Subject: " + subject + "\n\n")
	msg.WriteString(body)

	return smtp.SendMail(smtpHost+":"+smtpPort, auth, from, to, msg.Bytes())
}

func readEmailList(filePath string) ([]string, error) {
	file, err := os.Open(filePath)
	if err != nil {
		return nil, err
	}
	defer file.Close()

	var emailList []string
	scanner := bufio.NewScanner(file)
	for scanner.Scan() {
		email := strings.TrimSpace(scanner.Text())
		if email != "" {
			emailList = append(emailList, email)
		}
	}

	if err := scanner.Err(); err != nil {
		return nil, err
	}

	return emailList, nil
}

func main() {
	smtpHost := flag.String("smtpHost", "smtp.gmail.com", "SMTP server host")
	smtpPort := flag.String("smtpPort", "587", "SMTP server port")
	from := flag.String("from", "", "Sender email address")
	password := flag.String("password", "", "Sender email password")
	to := flag.String("to", "", "Recipient email address")
	subject := flag.String("subject", "Test Subject", "Email subject")
	body := flag.String("body", "This is the body of the email", "Email body")
	emailListPath := flag.String("emailList", "", "Path to a file containing a list of recipient email addresses")

	flag.Parse()

	if *from == "" || *password == "" || (*to == "" && *emailListPath == "") {
		flag.Usage()
		os.Exit(1)
	}

	var toEmails []string
	if *to != "" {
		toEmails = []string{*to}
	}
	if *emailListPath != "" {
		emails, err := readEmailList(*emailListPath)
		if err != nil {
			log.Fatalf("Failed to read email list: %v", err)
		}
		toEmails = append(toEmails, emails...)
	}

	err := sendEmail(*smtpHost, *smtpPort, *from, *password, toEmails, *subject, *body)
	if err != nil {
		log.Fatalf("Failed to send email: %v", err)
	}

	fmt.Println("Email sent successfully")
}

```

Compile the script: 

```
go build -o send-email send-email.go
```

Executing the script:

Single Recipient:
```
./send-email -from your-email@gmail.com -password your-password -to recipient@example.com -subject "Test Subject" -body "This is the body of the email"
```

Multiple Recipients: 

```
./send-email -from your-email@gmail.com -password your-password -emailList email_list.txt -subject "Test Subject" -body "This is the body of the email"
```