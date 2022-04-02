
   #[1]Medium [2]alternate

   [3]Get unlimited access
   [4]Open in app
   Home
   Notifications
   Lists
   Stories
     __________________________________________________________________

   Write

   Dhaura Pathirana
   [5]Dhaura Pathirana
   (BUTTON) Follow

   Mar 30
   �
   8 min read
   (BUTTON)
   (BUTTON)
   (BUTTON)
   (BUTTON)
   (BUTTON)
   (BUTTON)
   (BUTTON)
   (BUTTON)

Writing a Custom OAuth2 Grant Type in WSO2 Identity Server

   OAuth stands for Open Authorization and it is a lightweight protocol
   used for secure access delegation. It is a token-based access
   delegation method where a third-party application does not need the
   username and password of a user to access the user's resources.
   Currently, [6]OAuth 2.0 is the widely used version of OAuth.

OAuth2 Grant Types

   There are several ways or methods that can be used to implement the
   above-mentioned token-based authorization technique and these methods
   are known as "grant types". Four major standard grant types are
   supported in OAuth2 authorization servers and the WSO2 Identity Server
   has the flexibility to support any custom grant type as well. To learn
   more about these standard grant types, refer to the following blog.

Access Delegation with OAuth2

A brief introduction to OAuth 2.0 standard.

   dhaurapathirana.medium.com

Implementing a Custom Grant Type

   In order to implement a custom grant type, you need to write a handler
   and a validator for your grant type.
    1. Grant Type Handler -- Specifies how the validation must be done and
       how the token should be issued. This can be done in two ways.
       Either you can implement the "AuthorizationGrantHandler" Interface
       or you can extend the "AbstractAuthorizationGrantHandler" class. In
       this blog, the implementation will be done by extending the
       "AbstractAuthorizationGrantHandler" class.
    2. Grant Type Validator -- Verifies and validates the token request
       and checks whether all the required parameters are sent with the
       request. This can be implemented by extending the
       "AbstarctValidator" class.

   In this blog, for demonstration purposes, a simple custom grant type
   called "Mobile Grant" will be implemented. Here, you will pass a mobile
   number to the OAuth2 "token" endpoint in exchange for an access token.
   You can find the GitHub repository for the implementation, [7]here.

   First, a Maven Project should be created and in the source folder (a
   folder named "Java"), a package (org.wso2.carbon.identity.custom.grant)
   will be created for the new grant type. In this package, two java
   classes should be created. One for the validator and the other for the
   handler. For clarifications, the following image will illustrate the
   folder structure. [For relevant dependencies and imported packages, you
   can refer to the [8]pom.xml file in the GitHub repository.]
   Folder Structure for Mobile Grant

   Now let's move on to coding inside the java classes. First, let's go
   through the handler.

   In the MobileGrantHandler, the "AbstractAuthorizationGrantHandler"
   class will be extended as mentioned before. After that, the
   "validateGrant" method of the superclass will be overridden, so that,
   the validation for the new custom grant can be customized according to
   our needs.
@Override
public boolean validateGrant(OAuthTokenReqMessageContext tokReqMsgCtx)  throws I
dentityOAuth2Exception {

}

   This method will take an object of "OAuthTokenReqMessageContext" as a
   parameter that contains all the information that we send through the
   request.

   Inside the method, we'll first call the super "validateGrant" method to
   go through the default validation process and after that, we can start
   implementing our own validation.
boolean validateGrant = super.validateGrant(tokReqMsgCtx);

   If the result from the "super.validateGrant" is true, that means the
   default validation is successful. Then, we can move on to our
   validation process.

   In the token request, there will be three mandatory request parameters.
    1. Grant Type -- Specifies the type of the grant. (In this case, it'll
       be "mobile_grant")
    2. Mobile Number -- Number that is being used for the token request.
    3. ID Token -- It will be used to retrieve the user ID of the
       requester. (You can obtain an ID Token for a specific user by doing
       a Password Grant token request to the Identity Server with a scope
       of "openid")

   Next, the mobile number and the ID token will be extracted from the
   request parameters.
RequestParameter[] parameters = tokReqMsgCtx.getOauth2AccessTokenReqDTO().getReq
uestParameters();
String mobileNumber = null;
String userID = null;
String idTokenHint = null;
for (RequestParameter parameter : parameters) {
    if (MOBILE_NUMBER.equals(parameter.getKey())) {
        if (parameter.getValue() != null && parameter.getValue().length > 0) {
            mobileNumber = parameter.getValue()[0];
        }
    }
    if (ID_TOKEN_HINT.equals(parameter.getKey())) {
        if (parameter.getValue() != null && parameter.getValue().length > 0) {
            idTokenHint = parameter.getValue()[0];
        }
    }
}

   Here, MOBILE_NUMBER and ID_TOKEN_HINT are some constants that will be
   initialized at the beginning of the class.
public static final String MOBILE_NUMBER = "mobile_number";
public static final String ID_TOKEN_HINT = "id_token_hint";

   After that, a new method for mobile number validation will be declared.
private boolean isValidMobileNumber(String mobileNumber){
    // just demo validation
    if(mobileNumber.startsWith("011")){
        return true;
    }
    return false;
}

   This will be a private method since it will be only used inside the
   class. This is just a demo validation and you can customize this method
   as you need. Next, in the "validateGrant" method, it will be checked if
   the parameter values are not null and if the mobile number is valid
   according to the implemented validation method.

   As the next step, an Authorized user will be created and the user ID
   will be extracted from the ID token. Other user information will be
   hardcoded here for simplicity. (You can use the extracted user ID to
   obtain other user info but it would be a little bit complex for the
   purpose of this blog.) Another point that should be noted here is that
   you can use your own way to get this Authorized user according to the
   requirements of your custom grant type.
if (mobileNumber != null && idTokenHint != null && isValidMobileNumber(mobileNum
ber)){
    AuthenticatedUser mobileUser = new AuthenticatedUser();
    try {
        userID = SignedJWT.parse(idTokenHint).getJWTClaimsSet()
                .getSubject();
    } catch (ParseException e) {
        if (log.isDebugEnabled()) {
            log.debug("Error occurred while retrieving user ID from ID token", e
);
        }
    }
    mobileUser.setUserId(userID);
    mobileUser.setUserName(mobileNumber);
    mobileUser.setAuthenticatedSubjectIdentifier(mobileNumber);
    mobileUser.setFederatedUser(false);
    mobileUser.setTenantDomain("carbon.super");
    mobileUser.setUserStoreDomain("PRIMARY");
    tokReqMsgCtx.setAuthorizedUser(mobileUser);
tokReqMsgCtx.setScope(tokReqMsgCtx.getOauth2AccessTokenReqDTO().getScope());
    return true;
}

   In the end, the created authorized user will be inserted into the token
   request message context (tokReqMsgCtx) object. After that, if there are
   any scopes, they will also be inserted into the above object. If the
   whole process goes well, the "validateGrant" method will return true.
   (The whole code for "MobileGrantValidator" can be found in the
   previously mentioned GitHub repository.)

   Therefore, we can move on to the validator implementation.

   In the MobileGrantValidotor, you should extend the "AbstarctValidator"
   class as mentioned before. In the constructor, you should mention what
   parameters in the request body should be mandatory.
package org.wso2.carbon.identity.custom.grant;
import org.apache.oltu.oauth2.common.validators.AbstractValidator;
public class MobileGrantValidator extends AbstractValidator {
    public MobileGrantValidator() {
        requiredParams.add(MobileGrantHandler.MOBILE_NUMBER);
        requiredParams.add(MobileGrantHandler.ID_TOKEN_HINT);
    }
}

   Now, as the coding part has been concluded, we can do a Maven build
   with the project. First, open up a terminal and navigate into the
   project's root folder and enter the following command.
mvn clean install -DskipTests

   If the build is successful, a folder named "target" will be created and
   inside it, you'll find a [9].jar file for the mobile grant.
   Built .jar file for Mobile Grant

   After that, you can transfer (copy and paste) the above .jar file into
   IS_HOME/repository/components/lib folder. Here, IS_HOME is the root
   directory of the WSO2 Identity Server. (To download and install the
   Identity Server, follow this [10]link.)

   Now, open the IS_HOME/repository/conf/deployment.toml file and add the
   following lines at the end of the file.
[[oauth.custom_grant_type]]
name="mobile_grant"
grant_handler="org.wso2.carbon.identity.custom.grant.MobileGrantHandler"
grant_validator="org.wso2.carbon.identity.custom.grant.MobileGrantValidator"
[oauth.custom_grant_type.properties]
IdTokenAllowed=true

   After saving the above deployment.toml file, [11]run (or restart if
   already running) the Identity server.

   As the next step, a service provider application has to be created in
   the Identity Server and it can be done through the Management Console
   or just by executing the following curl command. ("mobile_grant" should
   be specified as an allowed grant type for the application.)
curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: app
lication/json" -d '{  "client_name": "application_test", "grant_types": ["passwo
rd","mobile_grant"], "redirect_uris":["http://localhost:8080/playground2"] }' "h
ttps://localhost:9443/api/identity/oauth2/dcr/v1.1/register"

   After the execution, you will get a response similar to the following.
{"client_name":"application_test","client_id":"vyMSzVoLS0ZUnY3F2gHkhytEdnka","cl
ient_secret":"Z4MYJLQyhCfrVE9mZBCRfO0C6Msa","redirect_uris":["<http://localhost:8>
080/playground2"]}

   Now, copy the client id and secret and get the base64 encoded value for
   client_id:client_secret. (Base64 encoded value can be obtained
   [12]here.) This value will be used in the Authorization header to
   authenticate the token request.

   To obtain an ID token, run the following command. (It's a password
   grant token creation request with OpenID as a scope.)
curl -k -X POST <https://localhost:9443/oauth2/token> -H 'Authorization: Basic <ba
se64encoded(client_id:client_secret)>' -H 'Content-Type: application/x-www-form-
urlencoded' -d 'grant_type=password&username=<username>&password=<password>&scop
e=openid'

   A sample request for the admin user is mentioned below.
curl -k -X POST <https://localhost:9443/oauth2/token> -H 'Authorization: Basic dnl
NU3pWb0xTMFpVblkzRjJnSGtoeXRFZG5rYTpaNE1ZSkxReWhDZnJWRTltWkJDUmZPMEM2TXNh' -H 'C
ontent-Type: application/x-www-form-urlencoded' -d 'grant_type=password&username
=admin&password=admin&scope=openid'

   In the response, you will receive an ID token and copy it for future
   use. (It will be used when writing a mobile grant token creation
   request.)

   Now, the grant type is ready for a test run. Let's create a Postman
   request for mobile grant token generation because Postman makes it
   easier to understand the request and its properties.

URL for the request:

<https://localhost:9443/oauth2/token>

Headers for the request:

   Under the "Authorization" key, the value should be replaced with the
   base64 encoded value that you've obtained before.

Body of the request:

   Here, the grant_type is specified as mobile_grant and a sample valid
   mobile number is supplied. There is only one scope is specified here
   but you can have a set of scopes here by space-separated. For the
   id_token_hint, replace the value with the ID token that you copied
   before.

   Now you can send the request and the response will be similar to the
   following.

   If you are able to get a response similar to the above response, then
   the custom grant type has been successfully implemented.

   Thus, we have reached the end of the blog, and hopefully, it would have
   been helpful for you to get started on creating a custom grant type
   with the WSO2 Identity Server.

References

Writing a Custom OAuth 2.0 Grant Type - WSO2 Identity Server Documentation

OAuth 2.0 authorization servers provide support for four main grant types
according to the OAuth 2.0 specification...

   is.docs.wso2.com

   (BUTTON) 15
   (BUTTON)

   (BUTTON) 15

   (BUTTON) 15
   (BUTTON)
   (BUTTON)

[13]More from Dhaura Pathirana

   (BUTTON) Follow
   [14](BUTTON)

   I am currently an intern software engineer at WSO2 and an undergraduate
   at University of Moratuwa following the degree, B.Sc. in CSE.

   Love podcasts or audiobooks? Learn on the go with our new app.
   [15]Try Knowable

More from Medium

   <img src="What-are-the-Trending-Android-App-Development-Ideas.png"
   alt="What are the Trending Android App Development Ideas?">
   What are the Trending Android App Development Ideas?
   Functional Automation Tests for XR applications
   ��[Big Update]_ WHAT DO USERS NEED TO PREPARE FOR TO RECEIVE NEW
   NFTs?��
   Build World-Class Blazor WebAssembly Apps with Globalization and
   Localization
   Build World-Class Blazor WebAssembly Apps with Globalization and
   Localization
   The missing CI/CD Kubernetes component: Helm package manager
   Becoming a Self-Taught Programmer: Cory Althoff Interview
   Day 54-- Validate BST
   100 Days to Amazon
   Containers over Virtual Machines
   Every other tech giant has shifted from virtual machines to
   containers .....well what made them do so??
   [16](BUTTON) Get started
   ____________________
   Dhaura Pathirana
   [17]

Dhaura Pathirana

   I am currently an intern software engineer at WSO2 and an undergraduate
   at University of Moratuwa following the degree, [18]B.Sc. in CSE.
   (BUTTON) Follow
   [19](BUTTON)

Related

   So you're looking for a package Manager?
   Why do I need a package manager?
   Can I build my own SAML / OAuth IDP?
   There are deluge of information about building SAML or OAuth platforms.
   In this post, I will quickly summarize the key considerations...
   Score Algorithm in ElasticSearch: BM25
   BM25 algorithm is known as the ranking algorithm in ElasticSearch and
   Lucene. If you request a "match" query, ElasticSearch engine will...
   Why Superior WebSocket Applications are Seriously Complicated and the
   Open Source Project I am...
   [20]

   Help
   [21]

   Status
   [22]

   Writers
   [23]

   Blog
   [24]

   Careers
   [25]

   Privacy
   [26]

   Terms
   [27]

   About
   [28]

   Knowable

References

   Visible links:

   1. <https://dhaurapathirana.medium.com/osd.xml>
   2. android-app://com.medium.reader/https/medium.com/p/6ab92881fdaf
   3. <https://medium.com/plans?source=upgrade_membership---nav_full>-------------------------------------
   4. <https://rsci.app.link/?$canonical_url=https%3A%2F%2Fmedium.com/p/6ab92881fdaf&~feature=LoOpenInAppButton&~channel=ShowPostUnderUser&~stage=mobileNavBar>
   5. <https://dhaurapathirana.medium.com/?source=post_page-----6ab92881fdaf>-----------------------------------
   6. <https://oauth.net/2/>
   7. <https://github.com/dhaura/identity-inbound-auth-oauth/tree/custom-grant/components/org.wso2.carbon.identity.custom.grant>
   8. <https://github.com/dhaura/identity-inbound-auth-oauth/blob/custom-grant/components/org.wso2.carbon.identity.custom.grant/pom.xml>
   9. <https://drive.google.com/drive/folders/1N6h1x5h4Py--9X3pWuLbhZ4sNxdoYhvm?usp=sharing>
  10. <https://is.docs.wso2.com/en/latest/setup/installing-the-product/>
  11. <https://is.docs.wso2.com/en/latest/setup/running-the-product/>
  12. <https://www.base64encode.org/>
  13. <https://dhaurapathirana.medium.com/?source=post_page-----6ab92881fdaf>-----------------------------------
  14. <https://medium.com/m/signin?actionUrl=%2F_%2Fapi%2Fusers%2F5c149789fd59%2Flazily-enable-writer-subscription&operation=register&redirect=https%3A%2F%2Fdhaurapathirana.medium.com%2Fwriting-a-custom-oauth2-grant-type-in-wso2-identity-server-6ab92881fdaf&user=Dhaura+Pathirana&userId=5c149789fd59&source=-----6ab92881fdaf---------------------subscribe_user>--------------
  15. <https://knowable.fyi/?utm_source=medium&utm_medium=referral&utm_campaign=medium-post-footer&source=post_page-----6ab92881fdaf>-----------------------------------
  16. <https://medium.com/m/signin?operation=register&redirect=https%3A%2F%2Fdhaurapathirana.medium.com%2Fwriting-a-custom-oauth2-grant-type-in-wso2-identity-server-6ab92881fdaf&source=post_page--------------------------nav_reg>--------------
  17. <https://dhaurapathirana.medium.com/>
  18. <http://B.Sc/>
  19. <https://medium.com/m/signin?actionUrl=%2F_%2Fapi%2Fusers%2F5c149789fd59%2Flazily-enable-writer-subscription&operation=register&redirect=https%3A%2F%2Fdhaurapathirana.medium.com%2Fwriting-a-custom-oauth2-grant-type-in-wso2-identity-server-6ab92881fdaf&user=Dhaura+Pathirana&userId=5c149789fd59&source=--------------------------subscribe_user>--------------
  20. <https://help.medium.com/hc/en-us>
  21. <https://medium.statuspage.io/>
  22. <https://about.medium.com/creators/>
  23. <https://blog.medium.com/>
  24. <https://medium.com/jobs-at-medium/work-at-medium-959d1a85284e>
  25. <https://policy.medium.com/medium-privacy-policy-f03bf92035c9>
  26. <https://policy.medium.com/medium-terms-of-service-9db0094a1e0f>
  27. <https://medium.com/about?autoplay=1>
  28. <https://knowable.fyi/>

   Hidden links:
  30. <https://medium.com/>
  31. <https://medium.com/>
  32. <https://medium.com/>
  33. <https://medium.com/m/signin?operation=register&redirect=https%3A%2F%2Fmedium.com%2Fme%2Fnotifications&source=--------------------------notifications_sidenav>--------------
  34. <https://medium.com/m/signin?operation=register&redirect=https%3A%2F%2Fmedium.com%2Fme%2Flists&source=--------------------------lists_sidenav>--------------
  35. <https://medium.com/m/signin?operation=register&redirect=https%3A%2F%2Fmedium.com%2Fme%2Fstories%2Fdrafts&source=--------------------------stories_sidenav>--------------
  36. <https://medium.com/m/signin?operation=register&redirect=https%3A%2F%2Fmedium.com%2Fnew-story&source=--------------------------new_post_sidenav>--------------
  37. <https://dhaurapathirana.medium.com/?source=post_page-----6ab92881fdaf>-----------------------------------
  38. <https://dhaurapathirana.medium.com/access-delegation-with-oauth2-a30837ce1e83>
  39. <https://is.docs.wso2.com/en/latest/learn/writing-a-custom-oauth-2.0-grant-type/>
  40. <https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fvote%2Fp%2F6ab92881fdaf&operation=register&redirect=https%3A%2F%2Fdhaurapathirana.medium.com%2Fwriting-a-custom-oauth2-grant-type-in-wso2-identity-server-6ab92881fdaf&user=Dhaura+Pathirana&userId=5c149789fd59&source=-----6ab92881fdaf---------------------clap_footer>--------------
  41. <https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fvote%2Fp%2F6ab92881fdaf&operation=register&redirect=https%3A%2F%2Fdhaurapathirana.medium.com%2Fwriting-a-custom-oauth2-grant-type-in-wso2-identity-server-6ab92881fdaf&user=Dhaura+Pathirana&userId=5c149789fd59&source=-----6ab92881fdaf---------------------clap_footer>--------------
  42. <https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fvote%2Fp%2F6ab92881fdaf&operation=register&redirect=https%3A%2F%2Fdhaurapathirana.medium.com%2Fwriting-a-custom-oauth2-grant-type-in-wso2-identity-server-6ab92881fdaf&user=Dhaura+Pathirana&userId=5c149789fd59&source=-----6ab92881fdaf---------------------clap_footer>--------------
  43. <https://ermablogs.medium.com/what-are-the-trending-android-app-development-ideas-ba84bd282aec?source=post_internal_links---------0>-------------------------------
  44. <https://medium.com/@sjaya/functional-automation-tests-for-xr-applications-eacf606fa0f3?source=post_internal_links---------1>-------------------------------
  45. <https://zukimoba.medium.com/%EF%B8%8F-%EF%B8%8F-big-update-what-do-users-need-to-prepare-for-to-receive-new-nfts-%EF%B8%8F-%EF%B8%8F-414f37d34a74?source=post_internal_links---------2>-------------------------------
  46. <https://medium.com/@rajeshwari.pandinagarajan/build-world-class-blazor-webassembly-apps-with-globalization-and-localization-cc315c756f2f?source=post_internal_links---------3>-------------------------------
  47. <https://gajus.medium.com/the-missing-ci-cd-kubernetes-component-helm-package-manager-1fe002aac680?source=post_internal_links---------4>-------------------------------
  48. <https://medium.com/@deltsova/becoming-a-self-taught-programmer-cory-althoff-interview-d447eb85829f?source=post_internal_links---------5>-------------------------------
  49. <https://akshay-ravindran.medium.com/day-54-validate-bst-2fc1259aa753?source=post_internal_links---------6>-------------------------------
  50. <https://bablisahu8983.medium.com/containers-over-virtual-machines-bfed95839546?source=post_internal_links---------7>-------------------------------
  51. <https://dhaurapathirana.medium.com/>
  52. <https://medium.com/@sinaijoshmeza/so-youre-looking-for-a-package-manager-7bd487cc778e?source=read_next_recirc---------0---------------------7237d20f_357f_45ac_bcc3_46cfde544772>----------
  53. <https://medium.com/@gokulnathb/can-i-build-my-own-saml-oauth-idp-4af08833d3ec?source=read_next_recirc---------1---------------------7237d20f_357f_45ac_bcc3_46cfde544772>----------
  54. <https://medium.com/@whitelatte46/score-algorithm-in-elasticsearch-bm25-6a01ac6577c3?source=read_next_recirc---------2---------------------7237d20f_357f_45ac_bcc3_46cfde544772>----------
  55. <https://medium.com/@uberscott/why-superior-websocket-applications-are-seriously-complicated-and-the-open-source-project-i-am-4f07aa24c232?source=read_next_recirc---------3---------------------7237d20f_357f_45ac_bcc3_46cfde544772>----------
