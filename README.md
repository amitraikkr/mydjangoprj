<h1> CI/CD Pipeline to automatically release code from GitHub to AWS Elastic Beanstalk application using AWS CodePipeline, 
and automatically generate issue in JIRA if code test & build stage failed</h1> 

<h2>Creating Django Application</h2>
<p> 1. Create Django application with a name ‘mydjangoapp”, you can use your own name. create a html page with text below. We will use this text later 
to fail the code pipeline build process<br>  
        <h4>GitHub - AWS CICD Pipeline Integration</h4>
Run command to create a requirements.txt file for all python dependencies required for application
		<h4>~/mydjangoapp$ pip freeze > requirements.txt</h4>
	Run server and check Django application is working using local URL<br>
<i>Click <a href="https://docs.djangoproject.com/en/3.1/intro/tutorial01"> here</a>, if you want to learn more about creating Django application.</i></p>
<p> 2. Create a folder <b>.ebextensions</b> and a config (django.config) file in this folder, add below code to file
<h4>option_settings:<br>
    aws:elasticbeanstalk:container:python:<br>
    WSGIPath: mydjangoproj.wsgi:application</h4>
</p>
<p> 3. Create a repository on GitHub and commit your Django application under that repository.</p>

<h2>Deploying Application on Elastic Beanstalk</h2>
<p> 4. Open command line interface and use eb cli commands to deploy Django application on Elastic Beanstalk. In your Django app folder 
run below command to create beanstalk environment, choose appropriate options 
<h4>~/mydjangoapp$ eb init</h4>
Run below command to deploy application
<h4>~/mydjangoapp$ eb create env-django-1</h4>
Once application is ready, use beanstalk URL to test the app it should show below message
			<h4>GitHub - AWS CICD Pipeline Integration</h4>
<i>Click <a href="https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3.html">here</a>, if you want to know more about eb cli and deploying application on Beanstalk
</i></p>

<h2>Create an AWS Code Pipeline</h2>
<p> 5. Create a buildspec.xml file in Django application and add below code that will perform a build test to check if index.html file contain CICD text or not (very simple test)
<h5>
version: 0.2<br>
phases: <br>
    install:<br>
        runtime-versions:<br>
            python: 3.8<br>
        commands:<br>
            - echo "installing django app on beanstalk"<br>
    pre_build:<br>
        commands: <br>
            - echo "Pre build phase of django app"<br>
            - pip install -r requirements.txt<br>
    build:<br>
        commands:<br>
            - echo "... build phase ..."<br>
            - echo "We will run a simple test to check text CICD"<br>
            - grep -Fq "CICD" ./djangoapp/templates/index.html<br>
    post_build:<br>
        commands:<br>
            - echo "Post build phase..."</h5>
</p>	
<p> 6. Login to AWS console and go to Code Pipeline to create a CICD pipeline to build, test and deploy the application code from GitHub to AWS Elastic Beanstalk<br>
      (i). In Source Stage choose Source provider as GitHub version 2 and create a Connection with GitHub using ‘Connect to GitHub’, Choose your Repository and Branch.
     (ii). In Build Stage, define a project with managed image ‘Amazon Linux 2’, choose new role and use the buildspec
    (iii). In Deploy Stage, choose Deploy provider as “AWS Elastic Beanstalk”, select your application and beanstalk environment that will be redeployed when GitHub repository will be updated.
<br>Create CodePipeline notifications on Pipeline setting > Notification page to trigger a SNS notification when a CodePipeline stage failed using new Target
You can initiate the CodePipeline by clicking the Release Change option Pipeline configuration screen
</p>

<h2>System Manager Parameter Store & Lambda Function</h2>

<p> 7.	Create Parameter Store for JIRA URL and authorization token, make sure authorization token is store as secureString. Parameter will have below name & values
        
        <h4>jiraurl: https://username.atlassian.net/rest/api/2/issue
        myAuthString: UprDvlIx9cbIxxxxxxxxxx</h4>
</p>
<p> 8. Now you have to create a lambda function that will be trigger by SNS notification when a Pipeline stage failed, 
Lambda function will call the JIRA API to create a ticket in JIRA project. Download the lambda.py file or copy & paste the code in lambda configuration screen to deploy. 
Add trigger to connect with SNS that was created in CodePipeline notification. </p>
	 
<h2>Testing the build failed to generate ticket in Jira</h2>
<p> 9. Update the index.html file and remove or update the CICD text since buildspec.yml file has a check to fail build if CICD is missing in index.html file. Commit your changes to GitHub. </p>
<p> 10.	GitHub will initiate the CodePipeline to deploy the new changes on Elastic Beanstalk application, but build stage will be failed due to missing text and will trigger the SNS notification that will later initiate the Lambda function. </p>
<p> 11.	Verify the JIRA project for a new issue that is created as a result of failed CodePipeline stage </p>

<h4>Terminate and delete all the resources to avoid any cost</h4>
