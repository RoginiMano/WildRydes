# WildRydes
Project : Serverless Web Application -AWS

Introduction: Wildrydes is a innovative project which makes the life simpler by choosing the fastest mode of transport.

AWS Services Used:
. Amplify
. Lambda
. DynamoDB
. IAM
. API Gateway
. CodeCommit

I would like to share step by step instructions,

Step1: Pull the source code and push it new repository.

* Select the region in which you want to work on.
* Create an empty repository.
* Add policy to IAM user so that I can use codecommit.
* Create Git Credentials to the IAM User to allow HTTPS connections to codecommit.
* Clone the repository(create an empty folder for future code)
* Copied the project code from aws s3 cp s3://ttt-wildrydes/wildrydes-site ./ --recursive because AWS locked or deleted the source code from their s3 bucket and then commit 
  it to new repo.

Step2: Host website and make updates to it.

* Goto Amplify ,select aws codecommit and select continue.
* Select the repository wildrydes-site from the drop down menu.
* On the Build settings page, leave all the defaults, select Allow AWS Amplify to automatically deploy all files hosted in the project root directory and choose Next.
* On the Review page select Save and deploy.
* The process took nearly 2-3 minutes for Amplify Console to create the necessary resources and to deploy the code.
* Once completed,select the link for master you'll see the build and deployment details related to the branch.

  ![Screenshot 2024-05-16 142434](https://github.com/RoginiMano/WildRydes/assets/164807520/96bd27dc-4de2-4964-8122-ec26ba843e8d)


  ![Screenshot 2024-05-15 115430](https://github.com/RoginiMano/WildRydes/assets/164807520/f1a48995-3ec0-4e9f-b34b-b7a0081ea8ea)

* Then just to make sure everything works I tried doing some changes in the code and it worked successfully.

Step3: User Registration and Login.

* Goto cognito and then use selected the attribute to login with their Email Address and selected no MFA since it is just a demo project.
* On the Integrate your app page, name your user pool: WildRydes. Under Initial app client, name the app client: WildRydesWebApp and keep the other settings as default.
* On the Review and create page, choose Create user pool.
* On the User pools page, select the User pool name to view detailed information about the user pool you created. Copy the User Pool ID in the User pool overview section and save it in a secure location on your local machine.
* Select the App Integration tab and copy and save the Client ID in the App clients and analytics section of your newly created user pool.
* Update the config.js file.

  window._config = {
    cognito: {
        userPoolId: 'us-west-2_kYxJ0J9Vw', // e.g. us-east-2_uXboG5pAb
        userPoolClientId: '7qt5a8rioor6uem7hpbsqe98tu', // e.g. 25ddkmj4v6hfsfvruhpfi7n4hv
        region: 'us-west-2' // e.g. us-east-2
    },
    api: {
        invokeUrl: '' // e.g. https://rc7nyt4tql.execute-api.us-west-2.amazonaws.com/prod',
    }
};

* In your terminal window, add, commit, and push the file to your Git repository to have it automatically deploy to Amplify Console.
* Open /register.html, or choose the Giddy Up! button on the homepage (index.html page) of your site.
* Complete the registration form and choose Let's Ryde. I used my own email id. Make sure to choose a password that contains at least one upper-case letter, a number, and a special character.
* You would receive a code to the email id which you used for registration.
* After updating the code then you would receive a notification for authentication make sure to copy it to your notepad.

   ![image](https://github.com/RoginiMano/WildRydes/assets/164807520/b50f7304-1402-457a-8ba0-243054bea601)

Step4: Ride Sharing information

* Create an Amazon dynamodb table.
* In the Overview tab > General Information section of your new table and choose Additional info. Copy the ARN. You will use this in the next section.
* Create an IAM role for the lambda function.
* Enter AWSLambdaBasicExecutionRole in the filter text box and press Enter.
* Select the checkbox next to the AWSLambdaBasicExecutionRole policy name and choose Next.
* Enter WildRydesLambda for the Role Name.
* Choose Create Role.
* In the filter box on the Roles page type WildRydesLambda and select the name of the role you just created.
* On the Permissions tab, under Add permissions, choose Create Inline Policy.
* In the Select a service section, type DynamoDB into the search bar, and select DynamoDB when it appears.
* Choose Select actions.
* In the Actions allowed section, type PutItem into the search bar and select the checkbox next to PutItem when it appears.
* In the Resources section, with the Specific option selected, choose the Add ARN link.
* Select the Text tab. Paste the ARN of the table you created in DynamoDB (Step 6 in the previous section), and choose Add ARNs.
* Choose Next.
* Enter DynamoDBWriteAccess for the policy name and choose Create policy.
* Create a lambda function by selecting Node.js 16.x for the Runtime.
* Select Use an existing role  and select WildRydesLambda.
* Scroll down to the Code source section and replace the existing code in the index.js code editor with the contents of requestUnicorn.js. The following code block displays the requestUnicorn.js file. Copy and paste this code into the index.js tab of the code editor.

const randomBytes = require('crypto').randomBytes;
const AWS = require('aws-sdk');
const ddb = new AWS.DynamoDB.DocumentClient();

const fleet = [
    {
        Name: 'Angel',
        Color: 'White',
        Gender: 'Female',
    },
    {
        Name: 'Gil',
        Color: 'White',
        Gender: 'Male',
    },
    {
        Name: 'Rocinante',
        Color: 'Yellow',
        Gender: 'Female',
    },
];

exports.handler = (event, context, callback) => {
    if (!event.requestContext.authorizer) {
      errorResponse('Authorization not configured', context.awsRequestId, callback);
      return;
    }

    const rideId = toUrlString(randomBytes(16));
    console.log('Received event (', rideId, '): ', event);

    // Because we're using a Cognito User Pools authorizer, all of the claims
    // included in the authentication token are provided in the request context.
    // This includes the username as well as other attributes.
    const username = event.requestContext.authorizer.claims['cognito:username'];

    // The body field of the event in a proxy integration is a raw string.
    // In order to extract meaningful values, we need to first parse this string
    // into an object. A more robust implementation might inspect the Content-Type
    // header first and use a different parsing strategy based on that value.
    const requestBody = JSON.parse(event.body);

    const pickupLocation = requestBody.PickupLocation;

    const unicorn = findUnicorn(pickupLocation);

    recordRide(rideId, username, unicorn).then(() => {
        // You can use the callback function to provide a return value from your Node.js
        // Lambda functions. The first parameter is used for failed invocations. The
        // second parameter specifies the result data of the invocation.

        // Because this Lambda function is called by an API Gateway proxy integration
        // the result object must use the following structure.
        callback(null, {
            statusCode: 201,
            body: JSON.stringify({
                RideId: rideId,
                Unicorn: unicorn,
                Eta: '30 seconds',
                Rider: username,
            }),
            headers: {
                'Access-Control-Allow-Origin': '*',
            },
        });
    }).catch((err) => {
        console.error(err);

        // If there is an error during processing, catch it and return
        // from the Lambda function successfully. Specify a 500 HTTP status
        // code and provide an error message in the body. This will provide a
        // more meaningful error response to the end client.
        errorResponse(err.message, context.awsRequestId, callback)
    });
};

// This is where you would implement logic to find the optimal unicorn for
// this ride (possibly invoking another Lambda function as a microservice.)
// For simplicity, we'll just pick a unicorn at random.
function findUnicorn(pickupLocation) {
    console.log('Finding unicorn for ', pickupLocation.Latitude, ', ', pickupLocation.Longitude);
    return fleet[Math.floor(Math.random() * fleet.length)];
}

function recordRide(rideId, username, unicorn) {
    return ddb.put({
        TableName: 'Rides',
        Item: {
            RideId: rideId,
            User: username,
            Unicorn: unicorn,
            RequestTime: new Date().toISOString(),
        },
    }).promise();
}

function toUrlString(buffer) {
    return buffer.toString('base64')
        .replace(/\+/g, '-')
        .replace(/\//g, '_')
        .replace(/=/g, '');
}

function errorResponse(errorMessage, awsRequestId, callback) {
  callback(null, {
    statusCode: 500,
    body: JSON.stringify({
      Error: errorMessage,
      Reference: awsRequestId,
    }),
    headers: {
      'Access-Control-Allow-Origin': '*',
    },
  });
}

Step5: Invoke ride sharing functionality.

* Create a new rest API.
* In the Settings section, enter WildRydes for the API Name and select Edge optimized in the Endpoint Type dropdown.
* Create Authorizer.
* Paste the Authorization Token copied from the ride.html webpage in the Validate your implementation section of Module 2 into the Authorization (header) field, and verify that the HTTP status Response code is 200.
* Create a new resource method.
* Deploy your API and copy the URL.
* Update the Website config.

  window._config = {

    cognito: {

        userPoolId: 'us-west-2_kYxJ0J9Vw', // e.g. us-east-2_uXboG5pAb         

        userPoolClientId: '7qt5a8rioor6uem7hpbsqe98tu', // e.g. 25ddkmj4v6hfsfvruhpfi7n4hv

        region: 'us-west-2' // e.g. us-east-2 

    }, 

    api: { 

        invokeUrl: 'https://g3ovfm6orc.execute-api.us-west-2.amazonaws.com/Dev' // e.g. https://rc7nyt4tql.execute-api.us-west-2.amazonaws.com/prod, 

    } 

};

* Validate the implementation.
* Update the ArcGIS JS version from 4.3 to 4.6 in the ride.html file as
* <script src="https://js.arcgis.com/4.6/"></script>
  <link rel="stylesheet" href="https://js.arcgis.com/4.6/esri/css/main.css">
* Visit /ride.html under your website domain.
* If you are redirected to the ArcGIS sign-in page, create a new sign in.
* After the map has loaded, click anywhere on the map to set a pickup location.
* Choose Request Unicorn. You should see a notification in the right sidebar that a unicorn is on its way and then see a unicorn icon fly to your pickup location.

  ![Screenshot 2024-05-15 132628](https://github.com/RoginiMano/WildRydes/assets/164807520/75203c56-2a9a-45e3-9c0f-8b4b991f879c)



 


