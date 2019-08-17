# Spring Security

### Table of Contents

  * [Security Basics](#Security-Basics)
	* [Spring Security Tutorial](#tutorial-summary)
  * [OAuth 2.0](#Queues)
	* [OAuth 2.0 Tutorial](#tutorial-summary-1)
  
## Security Basics

**What is authentication?**

Authentication means confirming your own identity. **Credential-based authentication** is about validating your credentials like User Name/User ID and password to verify your identity.

**What is authorization?**

Authorization means granting access to the system resources. Authorization occurs after your identity is successfully authenticated by the system.

## Spring Security Concepts

**Users:**

In Spring, **user** refers to the account associated with the person who uses the system, generally identified by unique usernames. The most common way to authenticate a user account is via password.

**Authorities:**

Spring uses the term authorities to describe the roles or groups to which user can belong. Various authorities can be granted to users. Authorities are arbitrary strings that have no inherent meaning. We get to define the authorities for our applications and we get to assign meaning to these authorities as we wish.

**Principal:**

The Principal is an object that represents a logged in user of the system. The Principal includes information such as the user's name and granted authorities.

**User Details Service:**

The User Details Service is the Spring Security component responsible for retrieving information about logged in users. We can create custom User Details Service components to fit whatever user repository our application uses, or use the default implementation if you use a standard user store and access it in a standard way.

**Authentication Manager:**

The Authentication Manager component is responsible for authenticating users in the system. It integrates with whatever user repository our application uses and has the ability to find users by username and find their associated authorities. Spring has some default implementations for common user repositories.

**Security Schema**

Spring Security supplies a standard SQL schema that can be used for user and authority data. This works well for projects that are being built from scratch or that don't already have a user repository. Spring Security makes it easy to incorporate this schema into the Authentication Manager for your application.

**Data Source:**

This provides database connection configuration if you choose to use the Spring Security default SQL schema.

**Password Encoder:**

A Password Encoder encodes incoming passwords before comparing them to the values in the database. The best choice for this if there are no other constraints is the **BCrypt encoder**. The following must be true for encoded passwords to work:
* Passwords must be encoded before being stored in the database.
* Incoming passwords must be encoded before being compared to values in the database.
* Incoming passwords and passwords in the database must be encoded using the same encoder.

**Endpoint Authorization:**
Spring Security applies authorization rules to individual or groups of endpoints (typically referred to as **sercured resources**). This is done by defining which authorities (typically referred to as **roles**) a user must have in order to access a given endpoint. We have the freedom to:
* Leave some endpoints open to everyone.
* Leave some endpoints open to all authenticated users.
* Limit access to some endpoints to only those user who have been granted particular authorities.

**Logout:**

Spring Security supports the ability to log users out of the system. We can define the following:
* The URL to be accessed when requesting to be logged out.
* The URL to which the user should be redirected upon successful logout.
* Cookies we would like to delete.
* Whether or not the HTTP Session should be invalidated on logout.

**Cross-Site Request Forgery (CSRF)**

According to OWASP, the definition of Cross Site Request Forgery attacks is: “Cross-Site Request Forgery (CSRF) is an attack that forces an end user to execute unwanted actions on a web application in which they're currently authenticated."
* Spring Security has CSRF protection turned on by default. 
* The CSRF protection is configurable so that it is compatible with a wide variety of UI/client technologies.

**Additional Resources:**

* [What is Ethical Hacking?](https://www.eccouncil.org/ethical-hacking/)
* [OWASP Top 10](https://www.owasp.org/images/7/72/OWASP_Top_10-2017_%28en%29.pdf.pdf)
* [SQL injection](https://www.w3schools.com/sql/sql_injection.asp)
* [Prepared Statements](https://www.w3schools.com/php/php_mysql_prepared_statements.asp)


### Tutorial: [Spring Security Tutorial](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/Tutorials/spring-security-tutorial.md)

### Tutorial Summary:

1.
1.
1.

## OAuth 2.0

**What is OAuth?**

OAuth (Open Authorization) is an open standard for token-based authentication and authorization in distributed web applications.

**Federated Authentication:**

This means that one system delegates the responsibility of authentication to another system. You have seen this with "Login with Facebook." or "Log in with your Google account." on various websites.

**Delegated Authorization:**

This involves you giving another user or an app permission to access one or more of the resources under your control. For example, you might have a task manager application that integrates with Google Calendar. This application may need permission to add events to your calendar.

## Spring OAuth Concepts

**Authorization Server:**

The authorization server is the part of an OAuth 2 system that issued the bearer tokens for users.

This is a stand alone Spring Boot service that serves OAuth 2 bearer tokens to clients. This service also validates tokens presented by a resource server and sends back the user data associated with the given token.

**Resource Server:**

A resource server is a service that contains endpoints that are protected by the bearer token. Several resource servers can share on authorization server. Authorization servers can also be resource servers. Adding the `@EnableResourceServer` annotation turns a web service into a resource server.

**Authentication Manager**

The Authentication Manager plays the same role in an OAuth 2 system as it does in a non-OAuth 2 Spring Security system. The authentication manager is used by the authorization server to authenticate users for incoming requests.

**Authorities:**

Authorities play the same role in an OAuth 2 system as the do in a non-OAuth 2 Spring Security System.

**password encoder:**

The password encoder is used to encode both incoming user password values and incoming client secrets before they are compared to the values stored in the system.

**Bearer Token:**

The access token used by OAuth 2. These tokens are arbitrary and have no meaning in and of themselves.

1. Bearer tokens are issued by the authorization server when the client requests a token and presents valid authentication data.
1. The client then uses the bearer token to access to a resource server.
1. Because these token have no inherent meaning, the resource server presents the token to the authorization server in exchange for user information.
1. The resource server grants access to the requested endpoint if the token is valid.

**Grant Types:**

OAuth 2 has different processes for granting tokens based on different use cases:
* Password Grant: the client exchanges a username/password for a token. This should only be used for granting access to your service's own application. The system should never allow third-party applications for gaining access to passwords.
* Authorization Code Grant: the client exchanges an authorization code for a token.
* Client Credential Grant: the application is requesting access as itself, not on behalf of a user.

**Clients:**

A client is a type of application that is allowed to access the system. For example:
* HTML5
* Android
* Desktop Application
* Smart Watch

The ability to control how both clients and users that can access the system allows much finer grained control over access. For example, you could grant one level of access to Joe Smith using an HTML5 application and another level to Joe Smith using his smart phone.

**Client Details Service:**

The client details service is the component responsible for retrieving information about the clients that are configured to have access to the system.

**Client Secret:**

The client secret is like a password for the given client. The client secret is set when the Client Details Service is being created, configured, and populated.

**User Info Endpoint:**

This is an endpoint on the authorization server that allows resource servers to exchange valid tokens for the associated user data.

### Tutorial: [OAuth 2 Tutorial](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/Tutorials/oauth-tutorial.md)

### Tutorial Summary:

1.
1.
1.

#### [Go Back...](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/README.md)