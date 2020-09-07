# O365-UC
Office 365 Account Registration Portal

## requirement
```bash
sudo apt-get install tmux python3 python3-pip
sudo pip3 install tornado js2py
```

## Installation
```bash
git clone https://github.com/HuJK/O365-UC.git
cd O365-UC/backend/

# Running in the foreground
python3 o365-creater_api.py

# Running in the backgroung
# I am used to use tmux. You can use screen dtach & etc.
tmux new -d -s o365 python3 o365-creater_api.py
```

## Usage

Then connect to [https://127.0.0.1:12536](https://127.0.0.1:12536) 

#### Create accounts by invite code:

![alt text](https://raw.githubusercontent.com/HuJK/O365-UC/master/Screenshots/14.PNG)
![alt text](https://raw.githubusercontent.com/HuJK/O365-UC/master/Screenshots/12.PNG)
![alt text](https://raw.githubusercontent.com/HuJK/O365-UC/master/Screenshots/13.PNG)

#### Configure invite code:

In the ```invite_code``` folder, each filename is a code and the content is the usage count.

Make sure these files only contain numbers. No any newline , or any characters other than ```[-0-9]```

Negtive number means this invite_code can redeem infinity times.

![alt text](https://raw.githubusercontent.com/HuJK/O365-UC/master/Screenshots/15.PNG)

If you want to use mysql instead of txt rw, or your own invite code check progress, please edit line 234 to line 268 at the ```backend/o365_creater_auth.py``` file:

Please return True or False

```python
    def check(self,password):
        if "." in password or "/" in password or "\\" in password:
            # Do not use '.' '\' '/' character in your invite code due to
            # Security concerns
            return False
        i_path = os.path.join(self.invite_code_path,password)
        if os.path.isfile(i_path):
            with open(i_path) as i_fileHendler:
                use_left = int(i_fileHendler.read())
            if use_left < 0:
                return True
            elif use_left == 0:
                os.remove(i_path)
                return False
            elif use_left == 1:
                os.remove(i_path)
                return True
            else:
                with open(i_path,"w") as i_fileHendler:
                    i_fileHendler.write(str(use_left - 1))
                return True
        return False
```

The following code will execute when user logout or 

```sid``` is the sesstion id. When user check pass, the system will generate one.

```self.loginUser[sid]["redeemed"]``` is whether the user created an account or not. 

The following function is check user redeemed or not. If user logout before they redeem, it will add use_left back.

```logout``` will be called when the user logout or other users login for all expired session.

```python
    def logout(self,sid):
        if "redeemed" in self.loginUser[sid] and self.loginUser[sid]["redeemed"] == False:
            i_path = os.path.join(self.invite_code_path,self.loginUser[sid]["invite_code"])
            if os.path.isfile(i_path):
                with open(i_path) as i_fileHendler:
                    use_left = int(i_fileHendler.read())
                if use_left >= 0:
                    with open(i_path,"w") as i_fileHendler:
                        i_fileHendler.write(str(use_left + 1))
            else:
                with open(i_path,"w") as i_fileHendler:
                    i_fileHendler.write(str(1))
        del self.loginUser[sid]
```



## Initial Setup:
 
##### Default password: ```admin```

![alt text](https://raw.githubusercontent.com/HuJK/O365-UC/master/Screenshots/01.PNG)
![alt text](https://raw.githubusercontent.com/HuJK/O365-UC/master/Screenshots/Setup.PNG)

Now, you can configure registration settings or CAPTCHA settings

#### Registration settings

![alt text](https://raw.githubusercontent.com/HuJK/O365-UC/master/Screenshots/09.PNG)

#### CAPTCHA:

It should be able to work at most of CAPTCHA services. 

Like google reCAPTCHA/hCAPTCHA/luosimao/腾讯云验证码. 

I only tested google reCAPTCHA v2 and hCAPTCHA.

###### google reCAPTCHA v2:

![alt text](https://raw.githubusercontent.com/HuJK/O365-UC/master/Screenshots/16.PNG)

###### hCAPTCHA:
![alt text](https://raw.githubusercontent.com/HuJK/O365-UC/master/Screenshots/17.PNG)

#### Admin password

![alt text](https://raw.githubusercontent.com/HuJK/O365-UC/master/Screenshots/10.PNG)
