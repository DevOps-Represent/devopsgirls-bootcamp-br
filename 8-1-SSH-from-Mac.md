# SSH access from Mac

### 1.) Open up your Terminal client

Open up **Finder**. Go to **Applications** > **Utilities**, then open up the app called **Terminal**

### 2.) Verify where your Key Pair is

By default, your Key Pair should be in your *Downloads* directory. Verify that it's there by pasting the following command:

```
ls ~/Downloads/[MY KEYPAIR NAME].pem
```

Make sure you replace *[MY KEYPAIR NAME]* with the name you gave your keypair. For example, if I made a Key Pair called `banana-smith-keypair`, then my command should look like this:

```
ls ~/Downloads/banana-smith-keypair.pem
```

### 3.) Change the ownership of the key pair file.

Remember: this keypair is your way of logging into your instance! Therefore, it needs to be accessible only by you. We do this by executing the following command:

```
chmod 600 ~/Downloads/[MY KEYPAIR NAME].pem
```

Again, you need to make sure that you replace *[MY KEYPAIR NAME]* with your Key Pair's name. So for `banana-smith-keypair`, we would paste:

```
chmod 600 ~/Downloads/banana-smith-keypair.pem
```

### 4.) Copy your instance IP or URL 

Remember how we made a note of the *Public IP* of your EC2 instance? Now we need to use it. Essentially, what we want to do is to login to the IP of the instance using the Key Pair we created. Put it in your clipboard, or make a note of it in Notepad.

### 5.) Login!

Finally, we login to the instance. We do this with a command that looks like this:

```
ssh ec2-user@[MY IP ADDRESS] -i ~/Downloads/[MY KEYPAIR NAME].pem
```

Assuming we have a *Public IP* of `22.22.22.22`, and we have a Key Pair called `banana-smith-keypair`, your command would look like this:

```
ssh ec2-user@22.22.22.22 -i ~/Downloads/banana-smith-keypair.pem
```


