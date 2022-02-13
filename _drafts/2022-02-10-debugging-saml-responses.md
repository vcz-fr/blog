---
title: "Debugging SAML responses"
categories: ["Development"]
tags: ["saml", "cloud", "aws", "cognito"]
---

- SAML IdP <-> Amazon Cognito diagram from AWS
- Explain steps
- Explain authority delegation and why this process is secure

- SAML response to the IdP, how to decode
https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true)XPath_expression('/samlp:Response/Assertion/Subject','%5C%5Cn')XML_Beautify('%5C%5Ct')

- SAML response schema, XMLNS exploration
- How to properly connect ADFS and Amazon Cognito
https://aws.amazon.com/blogs/mobile/building-adfs-federation-for-your-web-app-using-amazon-cognito-user-pools/



<!-- READ MORE -->

