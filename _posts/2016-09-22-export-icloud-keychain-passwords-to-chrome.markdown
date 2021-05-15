---
layout: post
title: Export iCloud Keychain passwords to Chrome
date: '2016-09-22 15:22:41'
permalink: "/export-icloud-keychain-passwords-to-chrome/"
tags:
- icloud-keychain
---

Apple provides us a unified way to store your passwords: iCloud Keychain. Using [Shared Web Credentials](https://developer.apple.com/reference/security/1654440-shared_web_credentials), iOS apps can take the advantage of iCloud Keychain as well. 

But what if you are using Chrome and Safari at the same time? If you use auto-generated password by Safari, you have to go to preferences and do some copy-and-paste job. And vice versa.  

After searching through all the sites, I found a way to achieve this, but still experiencing some issues. Safari stores passwords according to their domains, but Chrome stores passwords according to their exact filled-in URLs. That makes the synchronization nearly impossible. Personally I prefer the Safari-way. Chrome used to have a [flag](chrome://flags/#password-autofill-public-suffix-domain-matching), but in recent versions, they got disappeared. 

In this article, I will focus on how to export and import these passwords and left yourself dealing with the URL part. Same solution also works on Firebox, LastPass and 1Password. 

1. Download [keychaindump](https://github.com/juuso/keychaindump) from GitHub and compile it using OpenSSL

  `brew install openssl` 

  && 

  `gcc -I /usr/local/opt/openssl/include -L /usr/local/opt/openssl/lib keychaindump.c -o keychaindump -lcrypto -lssl`

2. Disable SIP if you are using El Capitan or later macOS. 

  Reboot with Command + R and enter `csrutil disable; reboot`

3. Create a new keychain and encrypted it with your **login password**, use copy / paste to populate the new keychain with all records from iCloud into the new keychain. Paste records into your new-created keychain. 

  **In my case, it is critical to store the new-created keychain under the default directory: `~/Library/Keychains/`**

  Then comes hardest part. Either one endures the maddening series of entering the keychain password and clicking 'OK' many times in a row, or editing and learning to use the AppleScript provided by another forum user. I chose the latter, and repeat the script here for completeness:

  <script src="https://gist.github.com/EvianZhow/1a809d3630eef193074838e79047b02b.js"></script>

  Open 'Script Editor' in macOS, paste the above code, replace `password` with your login password and play. 

4. Dump the passwords using **keychaindump**

  Open 'Keychain Access' and ensure your new-created keychain is unlocked. 

  `sudo keychaindump ~/Library/Keychains/NEW_KEYCHAIN_NAME.keychain`

5. Select / Save your passwords and process it with the underlying script. *(Optional)*

  <script src="https://gist.github.com/EvianZhow/dd8cf0cfe5fed3a89cebc242c4146408.js"></script>

  The CSV file is now suitable for importing. 

6. Enable Chrome password import feature by entering `chrome://flags/#password-import-export`, then restart the browser. 

7. Import password, re-enable SIP

  Reboot with Command + R and enter `csrutil enable; reboot`

8. Cleanup your plaintext password files!

### References

- [Import from iCloud-only keychain.txt](https://www.dropbox.com/sh/a3skeey2zqimdlv/AADarHXmCUkK8CRKLEDNdm-da/Extra%20Help/Import%20from%20iCloud-only%20keychain.txt?dl=0)
- [Doesn't work on OSX El Capitan?](https://github.com/juuso/keychaindump/issues/9)
- https://discussions.agilebits.com/discussion/comment/137713/#Comment_137713