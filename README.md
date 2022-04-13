---
url: "https://dhaurapathirana.medium.com/writing-a-custom-oauth2-grant-type-in-wso2-identity-server-6ab92881fdaf"
---

# blogs

A list of blogs.

lynx -force_html -nolist -dump "https://dhaurapathirana.medium.com/writing-a-custom-oauth2-grant-type-in-wso2-identity-server-6ab92881fdaf" > temp_doc;
temp_doc=$(lynx -force_html -nolist -dump "https://dhaurapathirana.medium.com/writing-a-custom-oauth2-grant-type-in-wso2-identity-server-6ab92881fdaf");
gramma listen -p "${temp_doc:200:20000}";
echo "${temp_doc:200:20000}";

doc_url=$(m2j -c README.md | jq -r .README.url);
temp_doc=$(lynx -force_html -nolist -dump $doc_url);
gramma listen -p "${temp_doc:200:20000}";

## get all links

lynx -force_html -listonly -dump "https://dhaurapathirana.medium.com/writing-a-custom-oauth2-grant-type-in-wso2-identity-server-6ab92881fdaf"
lynx -force_html -listonly -dump "https://wso2.com/community/"

m2j -c README.md | jq -r .README.url

gramma check -p README.md;

url=<https://wso2.com/community/>; blc $url -g --filter-level 1
