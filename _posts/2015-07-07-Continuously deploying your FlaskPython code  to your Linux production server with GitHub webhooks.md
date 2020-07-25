In this article I will describe how you can deploy your Flask or python code in your GitHub Repository to your Linux production server as soon as you have committed a change.

What is a GitHub Webhook?
GitHub Webhooks allow you to set triggers for specific events like a PUSH to your GitHub repository. When an event occurs GitHub will send a HTTP POST payload which contains data regarding your commit to the URL you have configured in your webhook settings.

To Continuously Deploy your Flask/Python code to your Linux production server you need to:



Write a Flask GitHub Webhook handler script that will pull changes to your production server when it receives the payload from GitHub.


Add a GitHub Webhook which will notify your script that an event such as PUSH has occurred on your GitHub repository.



Step 1: Creating and deploying the Flask GitHub WebHook handler script


#Compare the HMAC hash signature
def verify_hmac_hash(data, signature):
    GitHub_secret = bytes('some secret', 'UTF-8')
    mac = hmac.new(GitHub_secret, msg=data, digestmod=hashlib.sha1)
    return hmac.compare_digest('sha1=' + mac.hexdigest(), signature)
#Flask route that will process the payload
@app.route("/payload", methods=['POST'])
def GitHub_payload():
 signature = request.headers.get('X-Hub-Signature')
 data = request.data
 if verify_hmac_hash(data, signature):
   if request.headers.get('X-GitHub-Event') == "ping":
     return jsonify({'msg': 'Ok'})
   if request.headers.get('X-GitHub-Event') == "push":
      payload = request.get_json()
    if payload['commits'][0]['distinct'] == True:
      try:
         cmd_output = subprocess.check_output(['git', 'pull', 'origin', 'master'],)
         return jsonify({'msg': str(cmd_output)})
      except subprocess.CalledProcessError as error:
         return jsonify({'msg': str(error.output)})
 else:
   return jsonify({'msg': 'invalid hash'})
The complete code is available at https://GitHub.com/Leo-g/Flask-GitHub-Webhooks/blob/master/GitHub-webhook.py .

The first function in our GitHub Webhook Handler is a hash compare. While configuring a webhook you will need to set a  secret  which will be used to generate a hash signature. GitHub will send this hash signature along with each request in the headers as "X-Hub-Signature". Comparing the hash signature is the recommended way to secure your repository and ensure that the request came from your GitHub repository. GitHub uses a HMAC hexdigest to compute a hash signature, hence I have used the hmac module to generate and compare hashes, you can read more about the module and its methods in the docs at the end of the article.

Next I defined an endpoint for the URL where I want to receive the GitHub payload via Flask routes
“@app.route("/payload", methods=['POST'])” . A POST request to this URL will call the function GitHub_payload(). The function GitHub_payload() will first verify the hash signature and then return a response accordingly for the PING and PUSH events. In case of a PUSH event it will also run the “git pull “ command. The output of the command will be returned in the response. This will allow you to check if the pull was a success or not.

Deploying the script on your Linux production server

First Clone the script

   [leo@linux-production]$ git clone git@GitHub.com:Leo-g/Flask-GitHub-Webhooks-Handler.git

Next install flask

[leo@linux-production]$ cd    Flask-GitHub-Webhooks-Handler
[leo@linux-production Flask-GitHub-Webhooks]$ pip install -r requirements.txt         

Set your secret in an environment variable named GitHub_SECRET

[leo@linux-production Flask-GitHub-Webhooks]$ export GitHub_SECRET="some secret"

Add your IP address

[leo@linux-production Flask-GitHub-Webhooks]$ vim GitHub-webhook.py
 
 app.run(host="YourIP")

Run the script

[leo@linux-production Flask-GitHub-Webhooks]$ python3.4 GitHub-webhook.py 
 * Running on http://YourIP:5000/ (Press CTRL+C to quit)
 * Restarting with stat

Step 2: Adding a GitHub webhook to your repository

To add a GitHub WebHook to your repository you need to go to
https://GitHub.com/Leo-g/Flask-GitHub-Webhooks-Handler/settings/hooks (Replace Flask-GitHub-Webhooks-Handler  with the name of your repository)


 flask github webhook handler

Here you need to fill out the following fields:
Payload URL : Add “yourdomain.com/payload”
Content type : Select json
Secret : Add the same secret you used in your script to generate the HMAC hash.
Which events you like to trigger this webhook? : Select the push event for this tutorial.

Select the Active checkbox and click on Add webhook to start receiving notifications.

You should immediately receive a ping event.

If you received a “200 ok” response then you should be able to continuous deploy your GitHub code to your Linux production server in the event of a PUSH..( Make sure to add this script in the directory which contains your git repo information)

flask github webhook handler

Here is video of the whole process.

Video



References:
https://developer.GitHub.com/webhooks/securing/
http://githooks.com/
https://developer.GitHub.com/webhooks/
https://docs.python.org/3/library/hmac.html
