

<h1 style='text-align:center'>Email-Verification for Django</h1>

Email verification for new signups or new user is a two step verification process and adds a layer for security for valid users.

<b> verify_email </b> is a django app that provides this functionality right of the bat without any complex implementation.

## The app takes care of :
* Settings user's is_active status to False.
* Generate hashed token for each user.
* Generate a verification link and sending it to user's email.
* Recieve a request from verification link and verify for its validity.
* Activating the user's account.

## What you have to impliment is :
* Three steps in <a href='#quickstart'>Quick start</a> below...

<b>Note : </b>The app is designed to be used right of the bat, but further customizations options are also provided in <a href="#advance">Advance</a> section below.


## Installation

NOTE : Don't forget to activate virtual environment if you have one.

```
pip install Django-Verify-Email
```

<p id='quickstart'>
<h2>Quick start</h2> <hr>
</p>

The steps to getting started is very simple. Like any other app, this can be installed easyly by adding "verify_email" in your installed apps like:

<b>Note : </b>This documentation assumes that you already have a mail server configured for your project to send mails. 

if Not, then your first step should be Step 0:

### Step 0 :-

--- Bypass this step if you already have these things setup for your project. ---

In your settings.py :
```
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_PORT = 587
EMAIL_USE_TLS = True
EMAIL_HOST_USER = os.environ.get('EMAIL_ID') 
EMAIL_HOST_PASSWORD = os.environ.get('EMAIL_PW')

DEFAULT_FROM_EMAIL = 'noreply<no_reply@domain.com>'
```

## Main steps... <hr>

### Step 1 :-
Add "verify_email" to your INSTALLED_APPS setting like this:
```
    INSTALLED_APPS = [
        ...
        "verify_email",
    ]
```
<p id="step2">
<h3>Step 2 :-</h3>

Include the "verify_email" URLconf in your project urls.py like this:


```

urlpatterns = [
	...
	path('verification/', include('verify_email.urls')),	

]
```
</p>


### Step 3 :-
For sending email from a signup form, in your views.py import:

```
...
from verify_email.email_handler import send_verification_email
```
Now in the function where you are validation the form:

```
...

def register_user(request):
    ...
    
    if form.is_valid():

        inactive_user = send_verification_email(request, form)
```

<b>Attention : </b>"send_verification_email()" takes two arguments, requests and form in order to set user's active status. 

The "inactive_user" is returned by "send_verification_email()" contains a saved user object just like form.save() would do.(with is_active status set as False) which you can further user to extract user information from cleaned_data dict, like :

```
inactive_user.cleaned_data['email']

# Output : test-user123@gmail.com
```
The user is already been saved as inactive and you don't have to .save() it explicitly.

<b>If anything goes wrong in sending the verification link email, the user will not be saved, so that user can try again.</b>



### At this point, you are good to go...
 Start the development server and signup with an email and you should be getting an email on the entered email with the default template for account activation. (You can provide your own html template. see <a href='#advance'>Advance Section</a>)

 <b>Note : </b>The app comes with default email templates which can be overriden. See <a href='#customemailtemplate'> Custom Email Templates</a>

# Verifying User's email : 

<h3 style='text-align:center'>Nothing...</h3><br>

That's right ! , you don't have to impliment any other code for validating user with their respective unique tokens and emails. 

<b>The app takes care of everything in background.</b>

* When user click on the verification link, it comes to :
    ```
    path('verification/', include('verify_email')),
    ```
    which you defined in your project's urls.py in <a href='#step2'>step 2</a> above.
* This pattern is further extended in this app's urls.py where it accepts encoded email and encoded hashed token from the verification link.
* It then checks for user by that email.
* If user exist, it then checks for token if it is valid for that user or not.
* If the token is valid, it activates the user's account by setting is_active attribute to True and last_login to timezone.now().
* If the token is alredy been redeemed or modified, you'll be redirected to a verification failed page.

#### This whole process from generating HMAC hashed token for each user to verifying it for a unique user, is abstracted within the app 😃.



<p id="advance">

<h1>Advance</h1>

<p id="customemailtemplate">

<h2>Custom Email Templates : </h2>



The app is packed with default html templates to handle the web pages but if you want to provide your own template you can do it by setting an attribute in settings.py :

```
HTML_MESSAGE_TEMPLATE = "path/to/html_template.html"

VERIFICATION_SUCCESS_TEMPLATE = "path/to/success.html"

VERIFICATION_FAILED_TEMPLATE = "path/to/failed.html"
```
```
SUBJECT = 'subject of email'

# default subject is : Email Verification Mail
```
</p>

## Inside Templates : <hr>

### Custom HTML Message Template :

<b>IMPORTANT : </b> if you are using custom html message template for email that has to be sent to user, <u>provide a <b>{{link}}</b> as a template tag to contain verification link.</u> 

<b>You Must Pass This In Your Template</b> . Otherwise the sent mail will not contain the verification link.


For Ex :

```my_custom_email_message.html : ```

```
<div class="format-font" >
    <a href="{{link}}" class="my-button" >Verify</a>  # ----> The "link" variable is passed by the app's backend containing verification link.
</div>
```

----> "link" is a variable, that contains verification link, is passed in html message template during sending the email to user.


### Custom HTML Verification Success and Failed pages : 
<hr>

<b>Success :</b> 

Two variables are passed in context dict of "VERIFICATION_SUCCESS_TEMPLATE" :

* ```{{mgs}}``` : Which contains the message to be displayed on successful verification.
* ```{{link}}``` : Which contains redirect link to login page.

<b>In template :</b>

```
<h1 style="text-align: center; color: white;">
    {{msg}}     # __--> message variable
</h1>

<a href="{{link}}" class="btn btn-primary">     # __--> Link of login page
    Login
</a>

```

<b>Failed :</b>

Only "{{msg}}" is passed for failed msg in template.


<b>In template :</b>

```
<h1 style="text-align: center; color: white;">
    {{msg}}
</h1>
```



## Successful Verification :
After verification is successful, you might wanna redirect the user to login page. You can do this in two ways :

* 1 <b>Redirect from success webpage.</b>
	The user will be prompted to show success page with a button on it to navigate to Login page.
    ```
    LOGIN_URL = 'name of your login pattern'

    Note: This variable is also used by django.
    ```
* 2 <b>Redirect directly to login page without stopping at success message page.</b>
	The user will be directly sent to login page bypassing success page.
    ```
    VERIFICATION_SUCCESS_TEMPLATE = None
    ```
</p>





