# HTTPS for `local.planninglabs.nyc`

A set of scripts to quickly generate a HTTPS certificate for your local development environment.

## How-to

1. Clone this repository and `cd` into it:

```
git clone git@github.com:NYCPlanning/local-cert-generator.git
cd local-cert-generator
```
2. In this next step, you will run a script to create a root certificate, and it will ask you a series of questions. There is one crucial question that must be answered for using planninglabs subdomains locally:

![alt text](common-name.png)

Run this command with this caveat in mind:

```
sh createRootCA.sh
```

When prompted for the password, you can choose your own password, but make sure to write it down for adding/modifying the cert later.

For all questions except for Common Name, you can enter a period `.` to leave them blank. 

The command will output the rootCA file(s) into the local-cert-generator folder.

3. Add the root certificate we just generated (rootCA.pem) to your list of trusted certificates. This step depends on the operating system you're running:

    - **macOS**: Open Keychain Access and import the root certificate to your System keychain. 
      - Go to File > "Import Item"
      - After importing, you should see someting like this
      
      ![image](https://user-images.githubusercontent.com/3311663/78997227-2182bd00-7b14-11ea-8a9c-4012d3d4eb92.png)

    - Then mark the certificate as trusted.
      -  Double click the certificate, expand "trust", then select "Always Trust" for the "When using this certificate" prompt.

      ![image](https://user-images.githubusercontent.com/3311663/78997488-a968c700-7b14-11ea-8a39-a0c00d1e1722.png)


    ![Trust root certificate](https://cdn-images-1.medium.com/max/1600/1*NWwMb0yV9ClHDj87Kug9Ng.png)
    

    - **Linux**: Depending on your Linux distribution, you can use `trust`, `update-ca-certificates` or another command to mark the generated root certificate as trusted.

*Note*: You may need to restart your browser to load the newly trusted root certificate correctly.

4. Run the script to create a domain certificate for `local.planninglabs.nyc`: 

```
sh createSelfSigned.sh
```

5. Move `server.key` and `server.crt` to an accessible location on your server and include them when starting it. In an Express app running on Node.js, you'd do something like this, but it depends on the app in question:

```js
var path = require('path')
var fs = require('fs')
var express = require('express')
var https = require('https')

var certOptions = {
  key: fs.readFileSync(path.resolve('build/cert/server.key')),
  cert: fs.readFileSync(path.resolve('build/cert/server.crt'))
}

var app = express()

var server = https.createServer(certOptions, app).listen(443)
```

6. Make a copy of the subfolder that contains  `server.key` and `server.crt`, and place it under the client folder (keep the name used under the server folder). 
