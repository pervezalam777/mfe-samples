# Notes
## What are micro-frontends?
- Divide a monolithic app into multiple, smaller apps
- Each smaller app is responsible for a distinct feature of the product

## Why use them
- Multiple engineering teams can work in isolation
- Each smaller app is easier to understand and make changes to

## Integration
- There is no single perfect solution to integration
- Many solutions, each have pros and cons
- Look at what your requirements are, then pick a solution

## Major Categories of Integration
### Build-Time Integration
- Compile-Time Integration
- `Before` Container gets loaded in the browser, it gets access to ProductsList source code

#### Steps
- Engineering team develops ProductsList
- Publish ProductsList as an NPM package
- Team in charge of Container installs ProductsList as a dependency
- Container team builds their app
- Output bundle that includes all the code for ProductsList

#### Pros and Cons
- PROS: Easy to setup and understand!
- CONS: Container has to be re-deployed every time ProductsList is updated
- CONS: Tempting to tightly couple the Container + ProductsList together

### Run-Time Integration
- Client-Side Integration
- `After` Container gets loaded in the browser, it gets access to ProductsList source 

#### Steps
- Engineering team develops ProductsList
- ProductsList code deployed at https://my-app.com/productslist.js
- User navigates to my-app.com, Container app is loaded
- Container app fetches productslist.js and executes it

#### Pros and Cons
- PROS: ProductsList can be deployed independently at any time
- PROS: Different versions of ProductsList can be deployed and Container can decide which one to use 
- CONS: Tooling + setup is far more complicated 

### Server Integration
- While sending down JS to load up Container, a server decides on whether or not to include ProductsList source



# Instruction
## Working versions can be installed by running the following instead:
```console
npm install webpack@5.88.0 webpack-cli@4.10.0 webpack-dev-server@4.7.4 html-webpack-plugin@5.5.0 nodemon
```

## community fork was created to fix the issues which we can use instead:

instead of this:

      - uses: chrislennon/action-aws-cli@v1.1

write this:

      - uses: shinyinc/action-aws-cli@v1.2

This updated action will require an AWS_DEFAULT_REGION key, so, for now, we can just add a placeholder.

      - uses: shinyinc/action-aws-cli@v1.2
      - run: aws s3 sync dist s3://${{ secrets.AWS_S3_BUCKET_NAME }}/container/latest
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_DEFAULT_REGION: ""


Also, make sure to verify that you are using the correct branch name in your workflow. GitHub now prompts you to name this main by default.

## Key Creation Update + Reminder on AWS_DEFAULT_REGION
Update for Generating Keys
In the upcoming lecture, we will be creating an IAM user and then generating a key pair for deployment. There is a minor required change to this flow. Instead of being prompted to create a key pair during the IAM user creation, you must first create the IAM user, then, create a key pair associated with that user. AWS has also changed the terminology from Programmatic Access, to Command Line Interface (CLI).

Full updated instructions can be found below:

1. Search for "IAM"

2. Click "Create Individual IAM Users" and click "Manage Users"

3. Click "Add User"

4. Enter any name you’d like in the "User Name" field.

5. Click "Next"

6. Click "Attach Policies Directly"

7. Use the search bar to find and tick AmazonS3FullAccess and CloudFrontFullAccess

8. Click "Next"

9. Click "Create user"

10. Select the IAM user that was just created from the list of users

11. Click "Security Credentials"

12. Scroll down to find "Access Keys"

13. Click "Create access key"

14. Select "Command Line Interface (CLI)"


15. Scroll down and tick the "I understand..." check box and click "Next"

16. Copy and/or download the Access Key ID and Secret Access Key to use for deployment.

Reminder on AWS_DEFAULT_REGION
A few lectures ago we mentioned the need to use a different action and left a placeholder for the AWS_DEFAULT_REGION key.

Let's make sure this gets set correctly now.

In the AWS Dashboard use the Services search bar to find S3 and load its dashboard. Once there, copy the AWS region listed to the right of your bucket:


Then, in your container.yml, paste in the value for the AWS_DEFAULT_REGION like so:

      - uses: shinyinc/action-aws-cli@v1.2
      - run: aws s3 sync dist s3://${{ secrets.AWS_S3_BUCKET_NAME }}/container/latest
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_DEFAULT_REGION: us-east-2


## Minor Changes in AWS CloudFront UI
In the upcoming lecture, we will be creating our CloudFront distribution. The UI for this service has changed slightly, so, here are some notes to try and avoid confusion:

1. In the video there are two choices, Web and RTMP. Since the RTMP method has been removed, you no longer have to explicitly select Web as it is the default and only delivery method.

2. Distribution settings have been renamed to Settings.

3. The SSL certificate fields look different and no longer show that the default CloudFront certificate is selected. This is ok, the default certificate is still being used and nothing needs to be done or changed.

All other AWS settings and configurations should track what is shown in the videos through sections 7 and 8.

There is an AWS cheatsheet with all steps to create an S3 bucket, CloudFront distribution, and IAM user that will be regularly updated here:

https://www.udemy.com/course/microfrontend-course/learn/lecture/33274448#questions

## AWS Region with Automatic Invalidation
In the upcoming lecture, we will be adding automatic invalidation to our Container Workflow (and in a few lectures the Marketing Workflow).

If you are using the suggested shinyinc/action-aws-cli@v1.2 action, you will need to add the AWS Region to the variables associated with the create invalidation run step: 

- run: aws cloudfront create-invalidation --distribution-id ${{ secrets.AWS_DISTRIBUTION_ID }} --paths "/container/latest/index.html"
  env:
    AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
    AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    AWS_DEFAULT_REGION: us-east-2


Important - Remember to replace us-east-2 with whatever your actual region is.

## Small Required Change to historyApiFallback
Before starting on the next section, we will need to fix up a bug related to the historyApiFallback settings. Otherwise, you will be met with 404 errors in certain situations such as directly accessing http://localhost:8081/pricing.

Find this code in the webpack/dev.js file in both marketing and the container:

  devServer: {
    port: 8081,
    historyApiFallback: {
      index: "index.html",
    },
  },
You may resolve the issue by adding a / to the front of index.html:
```
  devServer: {
    port: 8081,
    historyApiFallback: {
      index: "/index.html",
    },
  },
```
Or, by setting to true:
```
  devServer: {
    port: 8081,
    historyApiFallback: true,
  },
```
After making this change, remember to restart both of your servers.

## Small Required Change to historyApiFallback
Similar to the previous section, we will need to fix up a bug related to the historyApiFallback settings. Otherwise, you will be met with 404 errors in certain situations such as directly accessing http://localhost:8082/auth/signup.

Find this code in the webpack/dev.js file of auth:
```
  devServer: {
    port: 8082,
    historyApiFallback: {
      index: "index.html",
    },
  },
```
You may resolve the issue by adding a / to the front of index.html:
```
  devServer: {
    port: 8082,
    historyApiFallback: {
      index: "/index.html",
    },
  },
```
Or, by setting to true:
```
  devServer: {
    port: 8082,
    historyApiFallback: true,
  },
```
After making this change, remember to restart your server.

