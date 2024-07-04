# AWS Cognito

Before discussing Amazon Cognito, it is first important to understand what Web Identity Federation is. 

**Web Identity Federation** lets you give your users access to AWS resources after they have successfully authenticated into a web-based identity provider such as Facebook, Google, Amazon, etc. 
Following a successful login into these services, the user is provided an auth code from the identity provider which can be used to gain temporary AWS credentials.

Amazon Cognito is the Amazon service that provides Web Identity Federation. You don’t need to write the code that tells users to sign in for Facebook or sign in for Google on your application. Cognito does that already for you out of the box.

Once authenticated into an identity provider (say with Facebook as an example), the provider supplies an auth token. This auth token is then supplied to cognito which responds with limited access to your AWS environment. You dictate how limited you would like this access to be in the IAM role.

![img](imgs/cognito.png)

To summarize, **Cognito's job is to broker between your app and legitimate authenticators**.

**Cognito User Pools** are user directories that are used for sign-up and sign-in functionality on your application. Successful authentication generates a JSON web token. Remember user pools to be user based. It handles registration, recovery, and authentication.

**Cognito Identity Pools** are used to allow users temp access to direct AWS Services like S3 or DynamoDB. Identity pools actually go in and grant you the IAM role.

SAML-based authentication can be used to allow AWS Management Console login for non-IAM users.
* You can use Microsoft Active Directory which implements Security Assertion Markup Language (SAML) as well.
* You can use Amazon Cognito to deliver temporary, limited-privilege credentials to your application so that your users can access AWS resources.
* Amazon Cognito identity pools support **both authenticated and unauthenticated identities**.
* You can retrieve a unique Amazon Cognito identifier (identity ID) for your end user immediately if you're allowing unauthenticated users or after you've set the login tokens in the credentials provider if you're authenticating users.
* When you need to easily add authentication to your mobile and desktop app, think Amazon Cognito.


## Cognito User Pools vs Identity Pools

**Cognito User Pools** are used for authentication. To verify your user’s identity, you will want to have a way for them to login using username/passwords or federated login using Identity Providers such as Amazon, Facebook, Google, or a SAML supported authentication such as Microsoft Active Directory. You can configure these Identity Providers on Cognito, and it will handle the interactions with these providers so you only have to worry about handling the Authentication tokens on your app.

With Cognito User Pools, you can provide sign-up and sign-in functionality for your mobile or web app users. You don’t have to build or maintain any server infrastructure on which users will authenticate. 

This diagram shows how **authentication** is handled with Cognito User Pools:

![imgs](imgs/Cognito-User-Pool-for-Authentication.png)

1. Users send authentication requests to Cognito User Pools. 
2. The Cognito user pool verifies the identity of the user or sends the request to Identity Providers such as Facebook, Google, Amazon, or SAML authentication (with Microsoft AD).
3. The Cognito User Pool Token is sent back to the user. 
4. The person can then use this token to access your backend APIs hosted on your EC2 clusters or in API Gateway and Lambda.  

If you want a quick login page, you can even use the pre-built login UI provided by Amazon Cognito which you just have to integrate on your application.

**Cognito Identity Pools** (Federated Identities) provides different functionality compared to User Pools. Identity Pools are used for User Authorization. You can create unique identities for your users and federate them with your identity providers. Using identity pools, users can obtain temporary AWS credentials to access other AWS services. 

Identity Pools can be thought of as the actual mechanism authorizing access to AWS resources. When you create Identity Pools, think of it as defining who is allowed to get AWS credentials and use those credentials to access AWS resources.

This diagram shows how **authorization** is handled with Cognito Identity Pools:

![imgs](imgs/Cognito-Identity-Pools-Federated-Identities.png)

1. The web app or mobile app sends its authentication token to Cognito Identity Pools. **The token can come from a valid Identity Provider, like Cognito User Pools, Amazon, Google, or Facebook**. 
2. Cognito Identity Pool exchanges the user authentication token for temporary AWS credentials to access resources such as S3 or DynamoDB. AWS credentials are sent back to the user. 
3. The temporary AWS credentials will be used to access AWS resources.   
You can define rules in Cognito Identity Pools for mapping users to different IAM roles to provide fine-grain permissions. 

Here’s a table summary describing Cognito User Pool and Identity Pool:

|Cognito User Pools | Cognito Identity Pools |
|---|---|
| Handles the IdP interactions for you|	Provides AWS credentials for accessing resources on behalf of users|
| Provides profiles to manage users	| Supports rules to map users to different IAM roles|
| Provides OpenID Connect and OAuth standard tokens| |
| Priced per monthly active user | Free |	


## AWS Security Token Service (STS)
AWS Security Token Service (AWS STS) is the service that you can use to create and provide trusted users with temporary security credentials that can control access to your AWS resources.

![img](imgs/sts.png)

* Temporary security credentials work almost identically to the long-term access key credentials that your IAM users can use.
* Temporary security credentials are short-term, as the name implies. They can be configured to last for anywhere from a few minutes to several hours. After the credentials expire, AWS no longer recognizes them or allows any kind of access from API requests made with them.

## Can you use Cognito for user management for your application, say a Facebook/Twitter clone? 

You absolutely can! Cognito User Pool is a serverless database of users for your web & mobile Apps.  
You get features like
1. Username (or email) / password combination
2. Password reset functionality
3. Email & Phone number Verification
4. Multi-factor Authentication
5. Federated Identities: users from Facebook, Google, SAML
6. Feature to Block users if their credentials are compromised elsewhere
7. Integrated with Services like API Gateway and Application Load Balancer (using listeners and rules)


# ID Token vs Access Token

When we sign in the Cognito user pool we will obtain all these three tokens. 
1. **The ID token** contains claims about the identity of the authenticated user such as name, email, and phone_number.
2. **The access token** contains scopes and groups and is used to grant access to authorized resources.
3. **The refresh token** contains the information necessary to obtain a new ID or access token.

### Which Token to Call API?
Normally, **the ID token is for Authentication, and the access token is for Authorization**. When calling the API method, we typically set the token to the request's Authorization header. So, it sounds like we should use the access token, doesn't it? Based on the doc, we could use either the ID token or the access token. However,  as we tested, we should use the ID token. That's because:

The ID token is used to authenticate users to our resource servers or server applications (ex: API)  
The purpose of the access token is to authorize API operations in the context of the user in the user pool. For example, you can use the access token to grant your user access to add, change, or delete user attributes.

# Control access to a REST API using Amazon Cognito user pools as authorizer

A common design scenario is to build a common security layer around the API gateway, so that all the APIs are secured. There are multiple ways to build API security like writing some filters in the case of Java / J2EE application, installing some agents in front of APIs which can make policy decisions etc. One of the most widely used protocol for Authorization is OAuth2. AWS API Gateway provides built-in support to secure APIs using AWS Cognito OAuth2 scopes.

![imgs](imgs/0_vXjRjS4vzOV9TFBh_.png)

this is how it works
1. Invoke AWS Cognito /oauth2/token endpoint with grant_type as client_credentials. Refer https://docs.aws.amazon.com/cognito/latest/developerguide/token-endpoint.html
2. If the request is valid, AWS Cognito will return a JWT (JSON Web Token) formatted access_token
3. Pass this token in Authorization header for all API calls
4. API Gateway makes a call to AWS Cognito to validate the access_token.
5. AWS Cognito returns token validation response.
6. If token is valid, API Gateway will validate the OAuth2 scope in the JWT token and ALLOW or DENY API call. This is entirely handled by API Gateway once configuration is in place
7. Perform the actual API call whether it is a Lambda function or custom web service application.
8. Return the results from Lambda function.
9. Return results to API Gateway.
10. If there are no issues with the Lambda function, API Gateway will return a HTTP 200 with response data to the client application.

# Using a Custom Authorizer 

In our API gateway, we wanted to validate some email domains before allowing access to the API calls, even if the users are authenticated by the cognito pool.

ie. even for users available+authenticated in the cognito user pool we wanted to restrict access to our api calls (authorization)

The two options available are
1. We validate the user in downstream lambda and based on the username/email return error
2. We use a custom lambda authorizer as discussed below

![imgs](imgs/1_l8FAm_sVT2b0Zvowz1DidA.png)

The problem with this change is that you need to reimplement what previous the Cognito authorizer did before and on top of that to put your logic. This is handled by the lambda (calling cognito in background for authenticating + custom authorization)

The lambda returns a policy which accepts or denies the API gateway access based.

We can enable Authorization Caching on this lambda, so that we dont trigger the lambda every time

