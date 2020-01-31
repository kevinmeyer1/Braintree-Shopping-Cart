# InClass03 Android App with API and Database

This is a class project from Advanced Mobile Application Development - Graduate course. The goal of this project is to build ontop of a previous project that my group created. We had a functioning shopping application where users could signup, login, fill up a shopping cart, and then check out. The only thing we were missing was the actual logic behind checking out. In the update of that project, this GitHub repository, I implemented Braintree into our mobile app and server to simulate real transactions.

This project includes the API and Android app needed to complete the requirements of InClass03.

The API was create in Node.js and uses Express for the routing. The data is stored on an Amazon AWS MySQL database.

The API is hosted on Heroku and can be accessed with a web address like: `https://inclass03.herokuapp.com/login`. This address specifically access the login route of the API.

## Routes

```/login```:

    https://inclass03.herokuapp.com/login

The login route takes in a username and password in a JSON object in the request body. The user name and password are verified to see if they exist in the database. If they do exist, the username is added as the payload to a JWT token. The JWT token is sent back to the app.

If authentication fails, a 'failure' response is sent back to the app and the app then handles that.

The payload of a JWT looks like this:

    {
        "username": "kevin"
    }
    
```/customer```:

    https://inclass03.herokuapp.com/customer
    
The customer route is used when a user tries to check out. The user's JWT token is sent to the server as the only part of the request body. The server then connects with Braintree and tries to find matching account information. If that information exists, a token used specifically by Braintree is returned otherwise the server goes through the process of creating a new customer on Braintree's end. Once everything is good to go, the Braintree token is returned back to the mobile application so that specific customer information, like saved credit cards, can be grabbed from the Braintree client within the app.

```/transaction```:

    https://inclass03.herokuapp.com/transaction 
    
This route is used when a user finally checks out and confirms that they want to pay for their items. It takes in a PaymentMethodNonce, a Braintree object that sensitive information about the actual transaction, and it also needs a payment amount. Once the route has that information, it simply connects to the Braintree APIs and completes a transaction. All of the information is then viewable on the Braintree dashboard. 
    
    
```/signup```:

    https://inclass03.herokuapp.com/signup

The signup route takes in a username, password, name, age, weight, and address in a JSON object in the request body. A query is sent to the database to create a new row with the given information. Username and password are the only required attributes so it is possible to the user to not supply name, age, weight, and address.

Once the user is created, a 'success' response is sent back to the app. The app then sends the user back to the login page with a message saying that the account has been created successfully. The user does not immediately get sent to their profile page once they create an account so there is no need to create or verify a JWT token in this route.

```/profile```:

    https://inclass03.herokuapp.com/profile

The profile route takes in only a JWT token as a JSON object in the request body. The JWT token is verified in the route. If it passes, the user's data is grabbed from the database using the JWT token to identify the username of the user. All the data is put into a JSON object and returned. If the verification fails, a 401 - Unauthorized error is sent back.

The returned JSON looks like this:

    {
        "name": "kevin",
        "age": "23",
        "weight": "200",
        "address": "100 Main Street"
    }

```/update_profile```:

    https://inclass03.herokuapp.com/update_profile

The update_profile route takes in the JWT token, username, name, age, weight, and address as a JSON object in the request body.

First, the route attempts to verify the JWT token sent. If the user is validated, an update query is sent to the database that will update the current users information. A new JSON object is then created with the users new information. The JSON is then used to create a new JWT token with an updated payload containing the user information. The token is sent back to the app.

If the user is not verified, 'failure' is sent back as the response and the app will handle it by forcing the user back to the login page since the user is not authenticated.

## Database Schema

The MySQL database on Amazon AWS is pretty basic.

| username | password | name | age | weight | address |
|----------|----------|------|-----|--------|---------|

All columns are VARCHARs. Username, password, and address are all max 100 characters. Age is max 3 characters and weight is max 4 characters. Username and password are the only required values, the other can be left blank if the user wants to.

## Assumptions Required to Provide Authentication

The only things required for authentication are a valid username and password that can successfully be found within the MySQL database. This will return a JWT token for that user. If there is already a JWT token stored on the device, the user does not have to do anything for authentication.

## Data Stored on Device

The JWT token is stored on the device once a user logs in. It is stored in the apps SharedPreferences. When the user logs out, the JWT token is removed. If a user logs in while there is a JWT saved to the device, they will be brought to the profile page as that user - the user they logged in as.
