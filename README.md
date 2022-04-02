---
url: "https://dhaurapathirana.medium.com/writing-a-custom-oauth2-grant-type-in-wso2-identity-server-6ab92881fdaf"
---

# blogs

A list of blogs.

lynx -dump URL "https://dhaurapathirana.medium.com/writing-a-custom-oauth2-grant-type-in-wso2-identity-server-6ab92881fdaf"

m2j -c README.md | jq -r .README.url

gramma listen -p $(lynx -verbose -dump $(m2j -c README.md | jq -r .README.url))
gramma listen -p "${"$(lynx -verbose -dump $(m2j -c README.md | jq -r .README.url))":200:20000}"

