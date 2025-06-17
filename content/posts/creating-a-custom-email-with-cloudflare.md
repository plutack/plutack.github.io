---
title: "Creating a Custom Email Address with Cloudflare Email Routing"
date: 2025-06-16T06:42:23+01:00
draft: false
toc: false
images:
tags: 
  - How-tos
---
  
A custom email address, such as `yourname@yourdomain.com`, significantly enhances your online presence and professional credibility, distinguishing you from generic email providers. While dedicated email hosting services like Zoho Mail or SendGrid are common solutions, Cloudflare offers a robust and often free alternative for managing custom email addresses, particularly if your primary need is forwarding inbound mail.

This guide will walk you through setting up a custom email address using Cloudflare Email Routing for incoming mail and configuring Gmail to send emails from your new custom address.
Requirements

To follow this guide, you will need:
- A custom domain name.
- Your domain's DNS managed by Cloudflare.

## Setting Up Email Forwarding (Receiving Mail)

Cloudflare Email Routing allows you to forward emails sent to your custom address to any destination inbox (e.g., your personal Gmail account).

1. Access Cloudflare Dashboard: Log in to your Cloudflare account and select the domain you wish to configure.
2. Navigate to Email Routing: In the left-hand sidebar, click on "Email" -> "Email Routing."
3. Create a Custom Address:
    - Click the "Create address" button.
    - In the "Custom address" field, enter the desired local part (the portion before the @ symbol) of your custom email, e.g., info for `info@yourdomain.com`.
    - In the "Destination address" field, enter the email address where you want to receive these forwarded emails (e.g., `yourpersonalemail@gmail.com`).
    - Click "Save."

At this stage, Cloudflare will automatically configure the necessary MX (Mail Exchange) records for your domain, ensuring that all emails sent to your newly created custom address are seamlessly forwarded to your specified destination address.


## Configuring Gmail to Send From Your Custom Address

While Cloudflare handles inbound mail forwarding, sending emails as your custom address requires an external SMTP server. This guide assumes you will use Gmail's SMTP server for outbound mail, allowing you to centralize your email management within your existing Gmail interface.
1. **Enable 2-Factor Authentication (2FA) for Your Google Account**

    Google requires 2FA to be enabled on your account before you can generate "App Passwords." App Passwords are essential for third-party applications (like Gmail's "Send mail as" feature) to authenticate without needing your primary Google password. If you haven't already, enable 2-Factor Authentication for your Google account: Google 2FA Setup.

2. **Generate an App Password**

    - After enabling 2FA, [Create a new app password](https://support.google.com/accounts/answer/185833?hl=en).

    - Under the "How you sign in to Google" section, click on "App passwords."

    - You may need to re-enter your Google password.

    - Enter a descriptive name (e.g., "Cloudflare Custom Email").

    Click "create" A 16-character alphanumeric password will be displayed. Copy this password immediately, as it will not be shown again.

3. **Add Your Custom Email to Gmail's "Send mail as"**

    - Open your Gmail inbox.

    - Click the Settings gear icon ⚙️ in the top right corner and select "See all settings."

    - Go to the "Accounts and Import" tab.

    - On the "Send mail as" section, click "Add another email address."

    -  A new window will appear:

        - Name: Enter the name you want recipients to see (e.g., "Your Name" or "Your Company").

        - Email address: Enter your full custom email address (e.g., `info@yourdomain.com`).

    Crucially, ensure you UNTICK "Treat as an alias." If this box is ticked, Gmail might send replies to your primary Gmail address instead of your custom one, and also impact how conversations are threaded. Unticking it ensures your custom address behaves as a separate sending identity.

    - Click "Next Step."

    - Fill out the SMTP server details:

        - SMTP Server: `smtp.gmail.com`

        - Port: `587`

        - Username: Your full Gmail address (e.g., yourpersonalemail@gmail.com).

        - Password: Paste the App Password you generated in the previous step.

        - Security: Ensure "Secured connection using TLS" is selected.

    - Click "Add Account."

    Gmail will send a confirmation email to your custom address (which will be forwarded by Cloudflare to your primary Gmail inbox). Open this email and click the confirmation link, or copy the confirmation code and paste it into the Gmail settings window.

Once confirmed, you will be able to select your custom email address from the "From" dropdown when composing new emails in Gmail.

## Enhancing Email Deliverability with SPF and DMARC

To improve the deliverability of emails sent from your custom address and protect your domain from spoofing, it's highly recommended to configure SPF (Sender Policy Framework) and DMARC (Domain-based Message Authentication, Reporting, and Conformance) records.
1. **Configure SPF Record**

SPF records specify which mail servers are authorized to send email on behalf of your domain. Since you're using Cloudflare's email routing and Gmail's SMTP, you need to include both.

  - In your Cloudflare dashboard, navigate to "DNS" -> "Records."

  - Click "Add record."
      - Type: TXT
      - Name: @
      - Content:
        ```
        "v=spf1 include:_spf.mx.cloudflare.net include:_spf.google.com ~all"
        ```
        include:_spf.mx.cloudflare.net: Authorizes Cloudflare's mail servers (for forwarding).

        include:_spf.google.com: Authorizes Google's mail servers (for sending via Gmail SMTP).

        ~all: A "softfail" policy, meaning emails from unauthorized servers might be accepted but marked as suspicious. Consider ~all over -all (hardfail) initially to avoid accidental rejections.

      - TTL: `Auto` 

  - Click "Save."

2. **Configure DMARC Policy**

DMARC builds upon SPF and DKIM (which Gmail handles for you) to provide a reporting and policy mechanism for email authentication failures.

  - In your Cloudflare dashboard, navigate to "Email" -> "DMARC Management."

  - Toggle DMARC management to "On."

  - Review the default DMARC record. You can edit the mailto: address to specify an email address where you want to receive reports on email authentication failures. This provides valuable insights into potential spoofing attempts or misconfigurations.

By implementing SPF and DMARC, you significantly reduce the likelihood of your emails being marked as spam and enhance the security posture of your domain.

## Conclusion

Leveraging Cloudflare Email Routing combined with Gmail's powerful sending capabilities provides an efficient and cost-effective solution for establishing a professional custom email address. This setup allows you to maintain the convenience of your existing Gmail inbox while presenting a polished, branded identity for your communications.

## Sources
  - [How to Generate & Use App Passwords](https://support.google.com/accounts/answer/185833?hl=en)
  - [How to use Gmail SMTP to send from an email address which uses Cloudflare Email Routing - Cloudflare Community](https://community.cloudflare.com/t/solved-how-to-use-gmail-smtp-to-send-from-an-email-address-which-uses-cloudflare-email-routing/382769)
  - [Github Gist: Cloudflare + Gmail for Custom Email](https://gist.github.com/irazasyed/a5ca450f1b1b8a01e092b74866e9b2f1#step-4-fill-out-the-next-form
)
