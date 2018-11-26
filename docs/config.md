This section provides the instructions to configure multi-factor authentication (MFA) using Email One Time Password (Email OTP) in WSO2 Identity Server (WSO2 IS). The Email OTP enables a one-time password (OTP) to be used at the second step of MFA. For more information on the WSO2 Identity Server Versions supported by the connector, see the IS Connector store.

Let's take a look at the tasks you need to follow to configure MFA using Email OTP:

- Enabling email configuration on WSO2 IS
- Configure the Email OTP provider
- Deploy the travelocity.com sample
- Configure the Identity Provider
- Configure the Service Provider
- Update the email address of the user
- Configure the user claims
- Test the sample

**Before you begin!**

   To ensure you get the full understanding of configuring Email OTP with WSO2 IS, the sample travelocity application is used in this use case. The samples run on the Apache Tomcat server and are written based on Servlet 3.0. Therefore, download Tomcat 7.x from here.
   Install Apache Maven to build the samples. For more information, see Installation Prerequisites.

**Enabling email configuration on WSO2 IS**

Follow the steps below to configure WSO2 IS to send emails once the Email OTP is enabled.

   1. Shut down the server if it is running.

   2. Open the  <IS_HOME>/repository/conf/axis2/axis2.xml file, uncomment the transportSender name = "mailto" 
   configurations, and update the following properties:
   
   |||
   |---|---|
   |**mail.smtp.from**|Provide the email address of the SMTP account.|
   |**mail.smtp.user**|Provide the username of the SMTP account.|
   |**mail.smtp.password**|Provide the password of the SMTP account.|

    <transportSender  name="mailto"
    class="org.apache.axis2.transport.mail.MailTransportSender">
        <parameter  name="mail.smtp.from">{SENDER'S_EMAIL_ID}</parameter>
        <parameter  name="mail.smtp.user">{USERNAME}</parameter>
        <parameter  name="mail.smtp.password">{PASSWORD}</parameter>
        <parameter  name="mail.smtp.host">smtp.gmail.com</parameter>
        <parameter  name="mail.smtp.port">587</parameter>
        <parameter  name="mail.smtp.starttls.enable">true</parameter>
        <parameter  name="mail.smtp.auth">true</parameter>
    </transportSender>

   3. Comment out the \<module ref="addressing"/> property to avoid syntax errors.
   
     <!-- <module ref="addressing"/> -->

   4.  Add the following email template to the <IS_HOME>/repository/conf/email/email-admin-config.xml.
    
    <configuration type="EmailOTP" display="idleAccountReminder" locale="en_US" emailContentType="text/html">
       <targetEpr></targetEpr>
       <subject>WSO2 IS Email OTP</subject>
       <body>
          Hi,
          Please use this one time password {OTPCode} to sign in to your application.
       </body>
       <footer>
          Best Regards,
          WSO2 Identity Server Team
          http://www.wso2.com
       </footer>
       <redirectPath></redirectPath>
    </configuration>

   5. Configure the following properties in the <PRODUCT_HOME>/repository/conf/identity/identity-mgt.properties file to
    true.

    Authentication.Policy.Enable=true
    Authentication.Policy.Check.OneTime.Password=true

   6. Add the following configuration to the <IS_HOME>/repository/conf/identity/application-authentication.xml file 
   under the <AuthenticatorConfigs> section.

    <AuthenticatorConfig name="EmailOTP" enabled="true">     
          <Parameter name="EMAILOTPAuthenticationEndpointURL">https://localhost:9443/emailotpauthenticationendpoint/emailotp.jsp</Parameter>
          <Parameter name="EmailOTPAuthenticationEndpointErrorPage">https://localhost:9443/emailotpauthenticationendpoint/emailotpError.jsp</Parameter>
          <Parameter name="EmailAddressRequestPage">https://localhost:9443/emailotpauthenticationendpoint/emailAddress.jsp</Parameter>
          <Parameter name="usecase">association</Parameter>
          <Parameter name="secondaryUserstore">primary</Parameter>
          <Parameter name="EMAILOTPMandatory">false</Parameter>
          <Parameter name="sendOTPToFederatedEmailAttribute">false</Parameter>
          <Parameter name="federatedEmailAttributeKey">email</Parameter>
          <Parameter name="EmailOTPEnableByUserClaim">true</Parameter>
          <Parameter name="CaptureAndUpdateEmailAddress">true</Parameter>
          <Parameter name="showEmailAddressInUI">true</Parameter>
    </AuthenticatorConfig>
   
   **Parameter definitions.**
   
   |Parameter|Description|
   |:---------:|:-----------:|
   |ssdsd   |sdsdfdsf|
   |dfsdf||
   
   7. Start WSO2 IS.

**Configure the Email OTP provider**

You can send the One Time Password (OTP) using Gmail APIs or using SendGrid. Follow the steps given below to configure Gmail APIs as the mechanisam to send the OTP.

   1. Create a Google account at [https://gmail.com](https://gmail.com).
    
   2. Got to [https://console.developers.google.com](https://console.developers.google.com) and click ENABLE APIS AND SERVICES.
    
   3. Search for Gmail API and click on it.

   4. Click Enable to enable the Gmail APIs.

   _Why is this needed?_

   _If you do not enable the Gmail APIs, you run in to a 401 error when trying out step13._


   5. Click Credentials and click Create to create a new project.

   6. Click Credentials and click the Create credentials drop-down.

   7. Select OAuth client ID option.
   8. Click Configure consent screen.
   9. Enter the Product name that needs to be shown to users, enter values to any other fields you prefer to update, 
   and click Save.

   10. Select the Web application option.
   11. Enter https://localhost:9443/commonauth as the Authorize redirect URIs text-box, and click Create.

    The client ID and the client secret are displayed.
    Copy the client ID and secret and keep it in a safe place as you require it for the next step.

   12. Copy the URL below and replace the <ENTER_CLIENT_ID> tag with the generated Client ID. This is required to 
   generate the authorization code.
   Format
   
   Example
   `https://accounts.google.com/o/oauth2/auth?redirect_uri=https%3A%2F%2Flocalhost%3A9443%2Fcommonauth&response_type
   =code&client_id=<ENTER_CLIENT_ID>&scope=http%3A%2F%2Fmail.google.com&approval_prompt=force&access_type=offline
   `
   12. Paste the updated URL into your browser.
       - Select the preferred Gmail account with which you wish to proceed.
       - Click Allow.
       - Obtain the authorization code using a SAML tracer on your browser.

   13. To generate the access token, copy the following cURL command and replace the following place holders:
       - **<CLIENT-ID>** : Replace this with the client ID obtained in Step 10 above.
       - **<CLIENT_SECRET>** : Replace this with the client secret obtained in Step 10 above.
       - **<AUTHORIZATION_CODE>** : Replace this with the authorization code obtained in Step 12 above.
   
   
   Format
       
         curl -v -X POST --basic -u <CLIENT-ID>:<CLIENT_SECRET> -H "Content-Type: application/x-www-form-urlencoded;charset=UTF-8" -k -d "grant_type=authorization_code&code=<AUTHORIZATION_CODE>&redirect_uri=https://localhost:9443/commonauth" https://www.googleapis.com/oauth2/v3/token
       
   Example
       
         curl -v -X POST --basic -u 854665841399-l13g81ri4q98elpen1i1uhsdjulhp7ha.apps.googleusercontent.com:MK3h4fhSUT-aCTtSquMB3Vll -H "Content-Type: application/x-www-form-urlencoded;charset=UTF-8" -k -d "grant_type=authorization_code&code=4/KEDlA2KjGtib4KlyzaKzVNuDfvAmFZ10T82usT-6llY#&redirect_uri=https://localhost:9443/commonauth" https://www.googleapis.com/oauth2/v3/token
     
   Sample Response
      
         curl -v -X POST --basic -u <CLIENT-ID>:<CLIENT_SECRET> -H "Content-Type: application/x-www-form-urlencoded;charset=UTF-8" -k -d "grant_type=authorization_code&code=<AUTHORIZATION_CODE>&redirect_uri=https://localhost:9443/commonauth" https://www.googleapis.com/oauth2/v3/token

   Paste the updated cURL command in your terminal to generate the OAuth2 access token, token validity period, and the refresh token. 


<\<IMAGE>>

   14. Update the following configurations under the  <AuthenticatorConfigs>  section in the  
   <IS_HOME>/repository/conf/identity/application-authentication.xml  file. 
        
        - If you need to send the content in a payload, you can introduce a property in a format <API> Payload and 
        define the value. Similarly, you can define the Form Data.FormdataforSendgridAPIisgivenasan example.
        You can use <API> URLParams, <API>AuthTokenType, <API>Failure and <API>TokenEndpoint property formats to specify the URL parameters, Authorization token type, Message to identify failure and Endpoint to get access token from refresh token respectively.
        Value of <API> URLParams should be like; api_user=<API_USER>&api_key=<API_KEY>&data=<DATA>&list<LIST>

   |Property|Description|
   |--------|-----------|
   |**GmailClientId**	|Enter the Client ID you got in step 10.Example: 501390351749-ftjrp3ld9da4ohd1rulogejscpln646s
   .apps.googleusercontent.com|
   |**GmailClientSecret**|	Enter the client secret you got in step 10.
    Example: dj4st7_m3AclenZR1weFNo1V|
   |**SendgridAPIKey**	|This property is only required if you are using the Sengrid method. Since you are using Gmail 
   APIs, keep the default value.|
   | **GmailRefreshToken**|	Enter the refresh token that you got as the response in step 12. Example: 
    1/YgNiepY107SyzJdgpynmf-eMYP4qYTPNG_L73MXfcbv|
   |**GmailEmailEndpoint**	|Enter your username of your Gmail account in place of the [userId] place holder. Example: 
   https://www.googleapis.com/gmail/v1/users/alex@gmail.com/messages/send|
   |**SendgridEmailEndpoint**|	This property is only required if you are using the Sengrid method. Since you are using 
   Gmail APIs, keep the default value.|
   |**accessTokenRequiredAPIs**	| Use the default value.|
   | **apiKeyHeaderRequiredAPIs**| This property is only required if you are using the Sengrid method. Since you are 
   using Gmail APIs, keep the default value.|
   | **SendgridFormData=to**|	This property is only required if you are using the Sengrid method. Since you are using 
   Gmail APIs, keep the default value.|
   | **SendgridURLParams**|	This property is only required if you are using the Sengrid method. Since you are using Gmail
    APIs, keep the default value.|
    |**GmailAuthTokenType**|	Use the default value.|
   | **GmailTokenEndpoint**|	Use the the deafult value.|
    |**SendgridAuthTokenType**|	This property is only required if you are using the Sengrid method. Since you are using 
    Gmail APIs, keep the default value.|
    
   Click here to see a sample configuration
   
     <AuthenticatorConfig name="EmailOTP" enabled="true">
        <Parameter name="GmailClientId">501390351749-ftjrp3ld9da4ohd1rulogejscpln646s.apps.googleusercontent.com 
       </Parameter>
        <Parameter name="GmailClientSecret">dj4st7_m3AclenZR1weFNo1V</Parameter>
        <Parameter name="SendgridAPIKey">sendgridAPIKeyValue</Parameter>
        <Parameter name="GmailRefreshToken">1/YgNiepY107SyzJdgpynmf-eMYP4qYTPNG_L73MXfcbv</Parameter>
        <Parameter name="GmailEmailEndpoint">https://www.googleapis.com/gmail/v1/users/alex@gmail
       .com/messages/send</Parameter>
        <Parameter name="SendgridEmailEndpoint">https://api.sendgrid.com/api/mail.send.json</Parameter>
        <Parameter name="accessTokenRequiredAPIs">Gmail</Parameter>
        <Parameter name="apiKeyHeaderRequiredAPIs">Sendgrid</Parameter>
        <Parameter name="SendgridFormData">sendgridFormDataValue</Parameter>
        <Parameter name="SendgridURLParams">sendgridURLParamsValue</Parameter>
        <Parameter name="GmailAuthTokenType">Bearer</Parameter>
        <Parameter name="GmailTokenEndpoint">https://www.googleapis.com/oauth2/v3/token</Parameter>
        <Parameter name="SendgridAuthTokenType">Bearer</Parameter>
        <Parameter name="redirectToMultiOptionPageOnFailure">false</Parameter>
     </AuthenticatorConfig>
   

**Deploy the travelocity.com sample**

Follow the steps below to deploy the travelocity.com sample application:

**Download the samples**

To be able to deploy a sample of Identity Server, you need to download it onto your machine first. 

Follow the instructions below to download a sample from GitHub.

    Create a folder in your local machine and navigate to it using your command line.

    Run the following commands.
    mkdir is-samples
    cd is-samples/
    git init
    git remote add -f origin https://github.com/wso2/product-is.git
    git config core.sparseCheckout true

    Navigate into the .git/info/ directory and list out the folders/files you want to check out using the echo command below.  
    cd .git
    cd info
    echo "modules/samples/" >> sparse-checkout

    Navigate out of .git/info directory and checkout the v5.4.0 tag to update the empty repository with the remote one. 
    cd ..
    cd ..
    git checkout -b v5.4.0 v5.4.0

    Access the samples by navigating to the  is-samples/modules/samples  directory.

**Deploy the sample web app**

Deploy this sample web app on a web container.

    Use the Apache Tomcat server to do this. If you have not downloaded Apache Tomcat already, download it from here. 
    Copy the .war file into the  webapps  folder. For example,  <TOMCAT_HOME>/apache-tomcat-<version>/webapps .
    Start the Tomcat server.

To check the sample application, navigate to http://<TOMCAT_HOST>:<TOMCAT_PORT>/travelocity.com/index.jsp on your browser.

For example, http://localhost:8080/travelocity.com/index.jsp.

Note: It is recommended that you use a hostname that is not localhost to avoid browser errors. Modify the /etc/hosts entry in your machine to reflect this. Note that localhost is used throughout thisdocumentation as an example, but you must modify this when configuring these authenticators or connectors with this sample application.


**Configure the Identity Provider**

Follow the steps below to add an identity provider:

    Click Add under Main > Identity > Identity Providers.
    Provide a suitable name for the identity provider.
    Expand the  EmailOTPAuthenticator Configuration under Federated Authenticators.

        Select the Enable and Default check boxes.

        Click Register.

    You have now added the identity provider.

**Configure the Service Provider**

Follow the steps below add a service provider:

    Return to the Management Console home screen.

    Click Add under Add under Main > Identity > Service Providers .

    Enter travelocity.com as the Service Provider Name.

    Click Register.

    Expand SAML2 Web SSO Configuration under Inbound Authentication Configuration.

    Click Configure.

    Now set the configuration as follows:

        Issuer: travelocity.com

        Assertion Consumer URL: http://localhost:8080/travelocity.com/home.jsp

        Select the following check-boxes: Enable Response Signing, Enable Single Logout, Enable Attribute Profile, and Include Attributes in the Response Always.

    Click Update to save the changes. Now you will be sent back to the Service Providers page.

    Go to Claim Configuration and select the http://wso2.org/claims/emailaddress claim.

    Go to Local and Outbound Authentication Configuration section.

        Select the Advanced configuration radio button option.

        Creating the first authentication step:

            Click Add Authentication Step.
            Click Add Authenticator that is under Local Authenticators of Step 1 to add the basic authentication as the first step.
            Adding basic authentication as a first step ensures that the first step of authentication will be done using the user's credentials that are configured with the WSO2 Identity Server

        Creating the second authentication step:

            Click Add Authentication Step.

            Click Add Authenticator that is under Federated Authenticators of Step 2 to add the SMSOTP identity provider you created as the second step.
            SMSOTP is a second step that adds another layer of authentication and security.

    Click Update.

    You have now added and configured the service provider.

    For more information on service provider configuration, see Configuring Single Sign-On.

[Back to Top]
Update the email address of the user

Follow the steps given below to update the user's email address.

    Return to the WSO2 Identity Server Management Console home screen.
    Click List under Add under Main > Identity > Users and Roles. 
        Click Users. 
        Click User Profile under Admin. 
        Update the email address.    
        Click Update.


**Configure the user claims**

Follow the steps below to map the user claims:

For more information about claims, see  Adding Claim Mapping. 

    Click Add under Main > Identity > Claims.

        Click Add Local Claim.
        Select the Dialect from the drop down provided and enter the required information.

        Add the following:
            Claim URI: http://wso2.org/claims/identity/emailotp_disabled
            Display Name: DisableEmailOTP
            Description: DisableEmailOTP
            Mapped Attribute (s): title

            Supported by Default: checked

        Click Add. 

        To disable this claim for the admin user, navigate to Users and Roles > List and click Users. Click on the User Profile link corresponding to admin account and then click Disable EmailOTP. This will disable the second factor authentication for the admin user.

**Test the sample**

    To test the sample, go to the following URL: http://localhost:8080/travelocity.com

    Click the link to log in with SAML from WSO2 Identity Server.

    The basic authentication page appears. Use your WSO2 Identity Server credentials.

    You receive a token to your email account. Enter the code to authenticate. If the authentication is successful, you are taken to the home page of the travelocity.com app.
