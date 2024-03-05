# SwiftSMTP

![SwiftSMTP bird](https://github.com/Kitura/Swift-SMTP/blob/master/Assets/swift-smtp-bird.png?raw=true)

> [!NOTE]
> This repository is a fork of [Kitura's Swift-SMTP](https://github.com/Kitura/Swift-SMTP).
>
> Since the project doesn't appear to be maintained anymore, this fork will contain bug fixes and features needed for the development of LocaleHub.
> 
> While we don't aim to revive the project, we will welcome any help from external contributors, and will reply to opened issues in pull requests in the best possible delays.

![macOS](https://img.shields.io/badge/os-macOS-green.svg?style=flat-square)
![Linux](https://img.shields.io/badge/os-linux-green.svg?style=flat-square)
[![Apache 2 license](https://img.shields.io/badge/license-Apache2-blue.svg?style=flat-square)](https://raw.githubusercontent.com/locale-hub/SwiftSMTP/master/LICENSE.txt)
[![Swift 5.2 and later](https://img.shields.io/badge/Swift-5.2-orange.svg?style=flat-square)](https://github.com/apple/swift)
[![Swift Package Manager compatible](https://img.shields.io/badge/Swift_Package_Manager-compatible-orange?style=flat-square)](https://swift.org/package-manager/)

## Features

Swift client for the SMTP protocol:
- Connect securely through SSL/TLS when needed
- Authenticate with CRAM-MD5, LOGIN, PLAIN, or XOAUTH2
- Send emails with local file, HTML, and raw data attachments
- Add custom headers
 
[Documentation](https://kitura.github.io/Swift-SMTP/)

## Swift Version

macOS & Linux: `Swift 5.2` or above.

## Installation

To install SwiftSMTP you can follow the [tutorial published by Apple](https://developer.apple.com/documentation/xcode/adding_package_dependencies_to_your_app)
using the URL for the SwiftSMTP repo with the current version:

1. In Xcode, select “File” → “Add Packages...”
1. Enter `https://github.com/locale-hub/SwiftSMTP.git`

or you can add the following dependency to your `Package.swift`:

```swift
.package(url: "https://github.com/locale-hub/SwiftSMTP.git", from: "6.0.1")
```

## Migration Guide

Version `5.0.0` brings breaking changes. See the quick migration guide [here](https://github.com/Kitura/Swift-SMTP/blob/master/migration-guide.md).

## Usage

Initialize an `SMTP` instance:

```swift
import SwiftSMTP

let smtp = SMTP(
    hostname: "smtp.gmail.com",     // SMTP server address
    email: "user@gmail.com",        // username to login
    password: "password"            // password to login
)
```

### TLS

Additional parameters of `SMTP` struct:

```swift
public init(hostname: String,
            email: String,
            password: String,
            port: Int32 = 587,
            tlsMode: TLSMode = .requireSTARTTLS,
            tlsConfiguration: TLSConfiguration? = nil,
            authMethods: [AuthMethod] = [],
            domainName: String = "localhost",
            timeout: UInt = 10)
```

By default, the `SMTP` struct connects on port `587` and sends mail only if a TLS connection can be established. It also uses a `TLSConfiguration` that uses no backing certificates. View the [docs](https://kitura.github.io/Swift-SMTP/) for more configuration options.

### Send email

Create a `Mail` object and use your `SMTP` handle to send it. To set the sender and receiver of an email, use the `User` struct:

```swift
let drLight = Mail.User(name: "Dr. Light", email: "drlight@gmail.com")
let megaman = Mail.User(name: "Megaman", email: "megaman@gmail.com")

let mail = Mail(
    from: drLight,
    to: [megaman],
    subject: "Humans and robots living together in harmony and equality.",
    text: "That was my ultimate wish."
)

smtp.send(mail) { (error) in
    if let error = error {
        print(error)
    }
}
```

Add Cc and Bcc:

```swift
let roll = Mail.User(name: "Roll", email: "roll@gmail.com")
let zero = Mail.User(name: "Zero", email: "zero@gmail.com")

let mail = Mail(
    from: drLight,
    to: [megaman],
    cc: [roll],
    bcc: [zero],
    subject: "Robots should be used for the betterment of mankind.",
    text: "Any other use would be...unethical."
)

smtp.send(mail)
```

### Send attachments

Create an `Attachment`, attach it to your `Mail`, and send it through the `SMTP` handle. Here's an example of how you can send the three supported types of attachments--a local file, HTML, and raw data:

```swift
// Create a file `Attachment`
let fileAttachment = Attachment(
    filePath: "~/img.png",          
    // "CONTENT-ID" lets you reference this in another attachment
    additionalHeaders: ["CONTENT-ID": "img001"]
)

// Create an HTML `Attachment`
let htmlAttachment = Attachment(
    htmlContent: "<html>Here's an image: <img src=\"cid:img001\"/></html>",
    // To reference `fileAttachment`
    related: [fileAttachment]
)

// Create a data `Attachment`
let data = "{\"key\": \"hello world\"}".data(using: .utf8)!
let dataAttachment = Attachment(
    data: data,
    mime: "application/json",
    name: "file.json",
    // send as a standalone attachment
    inline: false   
)

// Create a `Mail` and include the `Attachment`s
let mail = Mail(
    from: from,
    to: [to],
    subject: "Check out this image and JSON file!",
    // The attachments we created earlier
    attachments: [htmlAttachment, dataAttachment]
)

// Send the mail
smtp.send(mail)

/* Each type of attachment has additional parameters for further customization */
```

### Send multiple mails

```swift
let mail1: Mail = //...
let mail2: Mail = //...

smtp.send([mail1, mail2],
    // This optional callback gets called after each `Mail` is sent.
    // `mail` is the attempted `Mail`, `error` is the error if one occured.
    progress: { (mail, error) in
    },

    // This optional callback gets called after all the mails have been sent.
    // `sent` is an array of the successfully sent `Mail`s.
    // `failed` is an array of (Mail, Error)--the failed `Mail`s and their corresponding errors.
    completion: { (sent, failed) in
    }
)
```

## Acknowledgements

Inspired by [Hedwig](https://github.com/onevcat/Hedwig) and [Perfect-SMTP](https://github.com/PerfectlySoft/Perfect-SMTP).

## License

Apache v2.0
